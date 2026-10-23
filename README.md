# cake-teno-lib

TENO (Targeted Essentially Non-Oscillatory) convection schemes for OpenFOAM v2412, packaged as a drop-in library. It adds two schemes that stock OpenFOAM does not ship:

| Scheme | Name in fvSchemes | Stencils |
|---|---|---|
| TENO5 | teno | 3 candidate sub-stencils plus a wide central stencil |
| TENO6 | teno6 | 4 candidate sub-stencils plus a wide central stencil |

The schemes follow the two-level TENO reconstruction of Fu, Hu and Adams (2016), adapted to unstructured meshes with least-squares stencils. They are also bundled with [Cake Studio](https://github.com/CakeCFD/cake-studio), the open-source GUI for OpenFOAM.

## Status

The schemes as shipped are upwind-biased and second order. Faces are reconstructed from the upwind cell with a least-squares gradient, and near steep gradients the result is bounded by the neighbouring cell values. On smooth linear advection they measure about second order, matching linearUpwind in accuracy, not the fifth and sixth order of the reference formulation. Work toward the full design order is in progress.

| Scheme | Measured order (L1, N = 80 to 160) |
|---|---|
| linear (CD2) | 2.02 |
| linearUpwind | 1.81 |
| TENO5 | 1.88 |
| TENO6 | 1.88 |

The numbers come from the advection order test in the validation folder and are reproducible with:

```bash
source /usr/lib/openfoam/openfoam2412/etc/bashrc
python3 validation/tools/advection_order.py
```

## Install

### Prebuilt library

Each release has a prebuilt libtenoScheme.so for OpenFOAM v2412 (ESI) on Ubuntu 24.04, for x86_64 and aarch64. Download the one for your machine and place it in your user library folder:

```bash
source /usr/lib/openfoam/openfoam2412/etc/bashrc
mkdir -p "$FOAM_USER_LIBBIN"
cp libtenoScheme-v2412-$(uname -m).so "$FOAM_USER_LIBBIN/libtenoScheme.so"
```

### From source

```bash
source /usr/lib/openfoam/openfoam2412/etc/bashrc
git clone https://github.com/CakeCFD/cake-teno-lib.git
cd cake-teno-lib
wmake libso src
```

The library is written to $FOAM_USER_LIBBIN, which OpenFOAM searches automatically.

## Usage

Load the library in system/controlDict:

```
libs (libtenoScheme.so);
```

Select the scheme for the convection term in system/fvSchemes:

```
divSchemes
{
    default         none;
    div(phi,U)      Gauss teno;
}
```

Use teno6 in place of teno for the six-stencil variant.

## Example

The examples folder has a lid-driven cavity set up for icoFoam with TENO5:

```bash
source /usr/lib/openfoam/openfoam2412/etc/bashrc
cd examples/cavity
blockMesh
icoFoam
```

## Citation

If you use these schemes in published work, please cite this repository (see CITATION.cff) and the original TENO paper:

L. Fu, X. Y. Hu, N. A. Adams. A family of high-order targeted ENO schemes for compressible-fluid simulations. Journal of Computational Physics 305 (2016) 333-359. [doi:10.1016/j.jcp.2015.10.037](https://doi.org/10.1016/j.jcp.2015.10.037)

## License

GPL-3.0-or-later. See LICENSE.

## Trademark

This offering is not approved or endorsed by OpenCFD Limited, producer and distributor of the OpenFOAM software via www.openfoam.com, and owner of the OPENFOAM and OpenCFD trade marks.
