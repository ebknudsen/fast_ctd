project(
  'fast_ctd',
  'cpp',
  version: '0.1.0',
  meson_version: '>= 1.3.1'
)

# Compilers

cpp = meson.get_compiler('cpp')

# Imports

fs = import('fs')
py = import('python').find_installation()

# Useful paths

if get_option('PY_ENV_ROOT_PATH') == ''
  py_env_root = fs.parent(fs.parent(py.full_path()))
else
  py_env_root = get_option('PY_ENV_ROOT_PATH')
endif
py_env_lib = py_env_root / 'lib'
py_env_include = py_env_root / 'include'

message('Python environment root: ' + py_env_root)
if get_option('PY_ENV_ROOT_PATH') != ''
  message('(set by user)')
endif
message('Python environment lib: ' + py_env_lib)
message('Python environment include: ' + py_env_include)

# Deps and libs

nanobind_dep = dependency('nanobind')
spdlog = dependency('spdlog')
occ = dependency('OpenCASCADE')
moab = dependency('MOAB')
# Dependency('MOAB') fails to link properly when imported
# the library path has to be manually set
libmoab = cpp.find_library('MOAB', dirs: [py_env_lib])

# Run each subdir meson.build

subdir('src/occ_merger') # sets occ_merger_src, occ_merger_inc
subdir('src/occ_faceter') # sets occ_faceter_src, occ_faceter_inc

# Build the libraries that make up the extension

libocc_merger = static_library(
  'occ_merger',
  sources: [occ_merger_src],
  include_directories: [occ_merger_inc],
  dependencies: [occ, spdlog],
)
libocc_faceter = static_library(
  'occ_faceter',
  sources: [occ_faceter_src],
  include_directories: [occ_faceter_inc],
  dependencies: [occ, spdlog, moab, libmoab],
)

message('Building fast_ctd_ext and install python module')

# Build the extension module and install the Python source

py.extension_module(
  'fast_ctd_ext',
  sources: ['src/binding.cpp'],
  dependencies: [nanobind_dep],
  link_with: [libocc_merger, libocc_faceter],
  include_directories: [occ_merger_inc, occ_faceter_inc],
  install: true,
)
# v annoying that you have list each file individually
py.install_sources(
  [
    'fast_ctd/__init__.py',
    'fast_ctd/tools.py',
    'fast_ctd/utils.py',
  ],
  subdir: 'fast_ctd',
)
