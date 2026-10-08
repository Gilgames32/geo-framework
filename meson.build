project(
  'geo-framework',
  'cpp',
  version: '1.0',
  default_options: [
    'cpp_std=c++20',
    'warning_level=3',
  ],
  meson_version : '>= 1.1',
)

fs = import('fs')
cpp = meson.get_compiler('cpp')
cmake_exe = find_program('cmake')

cmake_builddir = meson.project_build_root() / 'cmake-build'
cmake_installdir = meson.project_build_root() / 'cmake-install'
cmake_libdirs = [cmake_installdir / 'lib']
cmake_includedir = cmake_installdir / 'include'

with_jet_fitting = get_option('with_jet_fitting')

cpp_args = [
  '-Wall',
  '-fPIC',
  '-Wno-reorder',
  '-D_USE_MATH_DEFINES',
]
if with_jet_fitting
  cpp_args += '-DUSE_JET_FITTING'
endif


qt6 = import('qt6')
qt6_dep = dependency('qt6', modules: [
  'Core',
  'OpenGL',
  'OpenGLWidgets',
  'Widgets',
  'Xml',
])

opengl_dep = dependency('gl')

openmesh_dep = dependency('OpenMesh', required: false)
# build openmesh from source if not found
if not openmesh_dep.found()
  om_src = meson.project_source_root() / 'subprojects/OpenMesh'
  om_builddir = cmake_builddir / 'OpenMesh'

  if not fs.exists(cmake_includedir / 'OpenMesh')
    run_command(cmake_exe, '-S', om_src, '-B', om_builddir,
      '-DCMAKE_BUILD_TYPE=Release',
      '-DBUILD_APPS=OFF',
      '-DOPENMESH_DOCS=OFF',
      '-DOPENMESH_BUILD_UNIT_TESTS=OFF',
      '-DCMAKE_POSITION_INDEPENDENT_CODE=ON',
      '-DCMAKE_INSTALL_PREFIX=' + cmake_installdir,
      check: true)
    run_command(cmake_exe, '--build', om_builddir, '--parallel', check: true)
    run_command(cmake_exe, '--install', om_builddir, check: true)
  endif

  openmesh_dep = declare_dependency(
    compile_args: ['-isystem', cmake_includedir],
    dependencies: [
      cpp.find_library('OpenMeshCore',  dirs: cmake_libdirs),
      cpp.find_library('OpenMeshTools', dirs: cmake_libdirs),
    ],
  )
endif

qglviewer_dep = dependency('QGLViewer', required: false)
# build qglviewer from source if not found
if not qglviewer_dep.found()
  qgl_src   = meson.project_source_root() / 'subprojects/libQGLViewer'
  qgl_builddir = cmake_builddir / 'libQGLViewer'

  if not fs.exists(cmake_includedir / 'QGLViewer')
    run_command(cmake_exe, '-S', qgl_src, '-B', qgl_builddir,
      '-DCMAKE_BUILD_TYPE=Release',
      '-DCMAKE_INSTALL_PREFIX=' + cmake_installdir,
      check: true)
    run_command(cmake_exe, '--build', qgl_builddir, '--parallel',
      '--target', 'QGLViewer',
      check: true)
    run_command(cmake_exe, '--install', qgl_builddir, check: true)
  endif

  qglviewer_dep = declare_dependency(
    compile_args: ['-isystem', cmake_includedir],
    dependencies: cpp.find_library('QGLViewer', dirs: cmake_libdirs),
  )
endif

deps = [qt6_dep, opengl_dep, openmesh_dep, qglviewer_dep]

if with_jet_fitting
  eigen3_dep = dependency('eigen3')
  cgal_dep = dependency('CGAL')
  deps += [eigen3_dep, cgal_dep]
endif

qt_headers = files(
  'viewer.hh',
  'window.hh',
)

qt_resources = files(
  'textures.qrc',
)

qt_sources = qt6.preprocess(
  moc_headers: qt_headers,
  qresources: qt_resources,
)

sources = files(
  'bezier.cc',
  'main.cc',
  'mesh.cc',
  'object.cc',
  'viewer.cc',
  'visualization.cc',
  'window.cc',
)
if with_jet_fitting
  sources += files('jet-wrapper.cc')
endif

sources += qt_sources

executable(
  'geo-framework',
  sources,
  dependencies: deps,
  include_directories: include_directories('.'),
  cpp_args: cpp_args,
  install: false,
)