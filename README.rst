lensfunpy
=========

A fork of lensfunpy that compatible with lensfun v0.3.4 and supports database v2.

lensfunpy is an easy-to-use Python wrapper for the lensfun library (v0.3.4).

`API Documentation <https://letmaik.github.io/lensfunpy/api/>`_

Sample code
-----------

How to find cameras and lenses
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    import lensfunpy

    cam_maker = 'NIKON CORPORATION'
    cam_model = 'NIKON D3S'
    lens_maker = 'Nikon'
    lens_model = 'Nikkor 28mm f/2.8D AF'

    db = lensfunpy.Database()
    cam = db.find_cameras(cam_maker, cam_model)[0]
    lens = db.find_lenses(cam, lens_maker, lens_model)[0]

    print(cam)
    # Camera(Maker: NIKON CORPORATION; Model: NIKON D3S; Variant: ; 
    #        Mount: Nikon F AF; Crop Factor: 1.0; Score: 0)

    print(lens)
    # Lens(Maker: Nikon; Model: Nikkor 28mm f/2.8D AF; Type: RECTILINEAR;
    #      Focal: 28.0-28.0; Aperture: 2.79999995232-2.79999995232; 
    #      Crop factor: 1.0; Score: 110)

How to correct lens distortion
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    import cv2 # OpenCV library

    focal_length = 28.0
    aperture = 1.4
    distance = 10
    image_path = '/path/to/image.tiff'
    undistorted_image_path = '/path/to/image_undist.tiff'

    img = cv2.imread(image_path)
    height, width = img.shape[0], img.shape[1]

    mod = lensfunpy.Modifier(lens, cam.crop_factor, width, height, focal_length)
    mod.enable_corrections(aperture=aperture, distance=distance, scale=0.0, flags=lensfunpy.ModifyFlags.VIGNETTING | lensfunpy.ModifyFlags.TCA)
    

    undist_coords = mod.apply_geometry_distortion()
    img_undistorted = cv2.remap(img, undist_coords, None, cv2.INTER_LANCZOS4)
    cv2.imwrite(undistorted_image_path, img_undistorted)

It is also possible to apply the correction via `SciPy <http://www.scipy.org>`_ instead of OpenCV.
The `lensfunpy.util <https://letmaik.github.io/lensfunpy/api/lensfunpy.util.html>`_ module
contains convenience functions for RGB images which handle both OpenCV and SciPy.

How to correct lens vignetting
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Note that the assumption is that the image is in a linear state, i.e., it is not
gamma corrected.

.. code-block:: python

    import lensfunpy
    import imageio

    db = lensfun.Database()
    cam = db.find_cameras('NIKON CORPORATION', 'NIKON D3S')[0]
    lens = db.find_lenses(cam, 'Nikon', 'Nikkor AF 20mm f/2.8D')[0]

    # The image is assumed to be in a linearly state.
    img = imageio.imread('/path/to/image.tiff')

    focal_length = 20
    aperture = 4
    distance = 10
    width = img.shape[1]
    height = img.shape[0]

    mod = lensfunpy.Modifier(lens, cam.crop_factor, width, height, focal_length)
    mod.enable_corrections(aperture=aperture, distance=distance, scale=0.0, flags=lensfunpy.ModifyFlags.VIGNETTING | lensfunpy.ModifyFlags.TCA)

    did_apply = mod.apply_color_modification(img)
    if did_apply:
        imageio.imwrite('/path/to/image_corrected.tiff', img)
    else:
        print('vignetting not corrected, calibration data missing?')


How to correct lens vignetting and TCA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Note that the assumption is that the image is in a linear state, i.e., it is not
gamma corrected. Vignetting should always be corrected first before applying the
TCA correction.

.. code-block:: python

    import imageio
    import cv2
    import lensfunpy

    db = lensfunpy.Database()
    cam = db.find_cameras('Canon', 'Canon EOS 5D Mark IV')[0]
    lens = db.find_lenses(cam, 'Sigma', 'Sigma 8mm f/3.5 EX DG circular fisheye')[0]

    # The image is assumed to be in a linearly state.
    img = imageio.imread('/path/to/image.tiff')

    focal_length = 8.0
    aperture = 11
    distance = 10
    width = img.shape[1]
    height = img.shape[0]

    mod = lensfunpy.Modifier(lens, cam.crop_factor, width, height, focal_length)
    mod.enable_corrections(aperture=aperture, distance=distance, scale=0.0, flags=lensfunpy.ModifyFlags.VIGNETTING | lensfunpy.ModifyFlags.TCA)

    # Vignette Correction
    mod.apply_color_modification(img)

    # TCA Correction
    undist_coords = mod.apply_subpixel_distortion()
    img[..., 0] = cv2.remap(img[..., 0], undist_coords[..., 0, :], None, cv2.INTER_LANCZOS4)
    img[..., 1] = cv2.remap(img[..., 1], undist_coords[..., 1, :], None, cv2.INTER_LANCZOS4)
    img[..., 2] = cv2.remap(img[..., 2], undist_coords[..., 2, :], None, cv2.INTER_LANCZOS4)

    imageio.imwrite('/path/to/image_corrected.tiff', img)

Installation
------------

Install lensfunpy by running:

.. code-block:: sh

    pip install lensfunpy

64-bit binary wheels are provided for Linux, macOS, and Windows.

Installation from source on Linux/macOS
---------------------------------------

If you have the need to use a specific lensfun version or you can't use the provided binary wheels
then follow the steps in this section to build lensfunpy from source.

First, install the lensfun_ library on your system.

On Ubuntu, install the lensfun v0.3.4 from the Git repository:

.. code-block:: sh

    git clone https://github.com/lensfun/lensfun
    cd lensfun
    git checkout tags/v0.3.4
    mkdir build
    cd build
    cmake ..
    sudo make install
    
After that, install lensfunpy using:

.. code-block:: sh

    git clone https://github.com/letmaik/lensfunpy
    cd lensfunpy
    pip install numpy cython
    pip install .
    
On Linux, if you get the error "ImportError: liblensfun.so.0: cannot open shared object file: No such file or directory"
when trying to use lensfunpy, then do the following:

.. code-block:: sh

    echo "/usr/local/lib" | sudo tee /etc/ld.so.conf.d/99local.conf
    sudo ldconfig

The lensfun library is installed in ``/usr/local/lib`` when compiled from source, and apparently this folder is not searched
for libraries by default in some Linux distributions.
Note that on some systems the installation path may be slightly different, such as ``/usr/local/lib/x86_64-linux-gnu``
or ``/usr/local/lib64``.

Installation from source on Windows
-----------------------------------

These instructions are experimental and support is not provided for them.
Typically, there should be no need to build manually since wheels are hosted on PyPI.

You need to have Visual Studio installed to build lensfunpy.

In a PowerShell window:

.. code-block:: sh

    $env:USE_CONDA = '1'
    $env:PYTHON_VERSION = '3.10'
    $env:PYTHON_ARCH = 'x86_64'
    $env:NUMPY_VERSION = '2.0.*'
    git clone https://github.com/letmaik/lensfunpy
    cd lensfunpy
    .github/scripts/build-windows.ps1

The above will download all build dependencies (including a Python installation)
and is fully configured through the four environment variables.
Set ``USE_CONDA = '0'`` to build within an existing Python environment.


.. _lensfun: https://lensfun.github.io/
