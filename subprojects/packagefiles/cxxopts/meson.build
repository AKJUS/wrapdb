project(
  'cxxopts',
  'cpp',
  version: '3.2.0',
  license: 'MIT',
)

install_headers('include/cxxopts.hpp')

pkgc = import('pkgconfig')
pkgc.generate(
  name: 'cxxopts',
  version: meson.project_version(),
  description: 'Lightweight C++ command line option parser',
)

cxxopts_dep = declare_dependency(
  include_directories: include_directories('include'),
)
