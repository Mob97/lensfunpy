# Lensfunpy Cython Wrapper Update Summary

## 1. **Database API Modernization**
- **Replaced deprecated/removed functions:**
  - `lf_db_new` → **`lf_db_create`**
  - `lf_db_load_file` → **`lf_db_load_path`** (and related public API)
  - `lf_db_find_lenses_hd` → **`lf_db_find_lenses`** (public API)

## 2. **Modifier API Modernization**
- **Replaced old modifier creation/initialization:**
  - Use **`lf_modifier_create`** instead of `lf_modifier_new`/`lf_modifier_initialize`
- **Correction enabling now uses:**
  - `lf_modifier_enable_distortion_correction`
  - `lf_modifier_enable_tca_correction`
  - `lf_modifier_enable_vignetting_correction`
  - `lf_modifier_enable_scaling`
  - `lf_modifier_enable_projection_transform`
  - `lf_modifier_enable_perspective_correction`

## 3. **Interpolation Functions**
- **Lens calibration interpolation now uses public API:**
  - `lf_lens_interpolate_distortion`
  - `lf_lens_interpolate_tca`
  - `lf_lens_interpolate_vignetting`
- **Removed/updated any use of legacy or private interpolation functions.**

## 4. **Cython Typing Fixes**
- **Function arguments like `float crop=None`** changed to `crop=None` (Python object), with explicit conversion to `cdef float` inside the function, to avoid Cython errors.

## 5. **General Clean-up**
- **Removed all usage and declarations of functions not present in the public `lensfun.h`.**
- **Ensured all Cython `cdef extern` blocks match the public API.**
- **Updated all usages in wrapper classes to use only public, supported Lensfun API.**

---

## ✅ Result

- The wrapper now builds and links cleanly against the latest Lensfun library.
- All database and modifier operations use the current, supported API.
- The code is compatible with Lensfun database version 2 and future-proofed for new releases.

---

If you need a more detailed changelog or want to see the specific code snippets, let me know!
