# Qt to GTK Repository Migration Standard

Use this standard when a project has moved from Qt to GTK and the GTK implementation
is the maintained application. It keeps the active code easy to find while preserving
the Qt implementation as an explicit historical archive.

## Target layout

The repository root must represent the active GTK application.

```text
project/
├── src/                 # GTK application and non-UI C++ code
├── data/                # GTK resources, CSS, desktop data, and templates
├── tests/               # GTK and shared tests
├── packaging/           # RPM spec, desktop entry, metainfo, and icons
├── docs/                # Current project documentation
├── legacy/
│   └── qt/              # Archived Qt implementation and Qt-era material
├── CMakeLists.txt       # Builds GTK directly
├── README.md
└── Makefile
```

Do not leave the maintained GTK code inside a wrapper such as `gtk/`. A contributor
should be able to open the repository and find the current application in `src/`.

## Archive the Qt implementation

Move every Qt-only item into `legacy/qt/`, retaining its relative structure where
practical:

| Item | Archive location |
| --- | --- |
| Qt source tree | `legacy/qt/src/` |
| Qt resources and `.qrc` files | `legacy/qt/resources/` |
| Qt-only screenshots | `legacy/qt/screenshots/` |
| Qt plans, design notes, and old setup instructions | `legacy/qt/docs/` |

Add `legacy/qt/README.md` stating that the code is historical reference only and
is not built, tested, packaged, or eligible for new feature work.

Keep release history in the root changelog. Historical Qt entries are valid there
because the changelog records released behaviour.

## Promote GTK build files

Move the GTK CMake configuration into the root `CMakeLists.txt`. The root build must:

1. Declare the project version and C++ standard.
2. Find GTK, libadwaita, JSON-GLib, and GResource tooling.
3. Compile resources from root-relative `data/` paths.
4. Build the application from `src/`.
5. Build tests from `tests/` when `BUILD_TESTING` is enabled.
6. Install the binary, desktop entry, metainfo, and icons.

Remove `add_subdirectory(gtk)` and any paths that treat GTK as a subproject. The
documented build must remain:

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTING=ON
cmake --build build -j$(nproc)
ctest --test-dir build --output-on-failure
```

## Packaging and CI

The source archive, source RPM, binary RPM, and CI workflow must build only the
GTK implementation.

- Update the source archive manifest to include `src/`, `data/`, `tests/`, current
  documentation, and packaging files.
- Exclude `legacy/qt/` from source archives and source RPMs.
- Remove Qt build dependencies from the RPM spec and CI environment.
- Update spec comments to describe the root GTK build.
- Keep CI commands rooted at `cmake -S .` and `ctest --test-dir build`.

Archiving the Qt tree in Git does not require shipping it to package builders.

## Documentation requirements

Update the README and contributor documentation to state:

- `src/`, `data/`, and `tests/` are the maintained GTK layout.
- The root CMake configuration builds the application.
- `legacy/qt/` is a read-only historical reference.
- Qt is not a build, test, runtime, or packaging dependency.

Update internal project memory, development notes, and path references at the same
time. Do not leave active documentation referring to `gtk/src/`, a Qt-only build,
or obsolete Qt classes and tools.

## Generated build directories

Build directories are disposable outputs, not migration artifacts. Before validating
the new layout, remove stale top-level build directories such as `build/`,
`build-gtk/`, and other `build-*` directories. Then configure a fresh build directory
from the repository root.

Keep generated build directories ignored by Git. Do not move them into `legacy/qt/`
or include them in source archives.

## Required validation

Before committing, complete these checks:

| Check | Expected result |
| --- | --- |
| Fresh configure | `cmake -S . -B build-check` succeeds. |
| Build | The GTK binary builds from root paths. |
| Tests | `ctest --test-dir build-check --output-on-failure` passes. |
| Source archive | Contains `src/`, `data/`, and `tests/`; excludes `legacy/qt/`. |
| RPM build | Source RPM and binary RPM build successfully. |
| Installed metadata | Desktop-file and AppStream validation pass during the RPM build. |
| Reference audit | Active CMake, README, packaging, CI, and docs contain no stale Qt or `gtk/` source paths. |

Use `git diff --check` before staging. Review file moves as renames where Git can
detect them, then stage the archive and promoted GTK files together so the repository
never has two competing active source layouts.
