# Copyright © 2017 Dylan Baker

# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:

# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.

# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

project(
  'glew',
  'c',
  version: '2.2.0',
  license: 'MIT AND BSD-3-Clause',
  meson_version: '>= 0.49.0',
)

cc = meson.get_compiler('c')
host_system = host_machine.system()

dep_gl = dependency('gl')

dep_glu = dependency(
  'glu',
  required: get_option('glu'),
)
if not dep_glu.found()
  add_project_arguments(
    '-DGLEW_NO_GLU',
    language: 'c',
  )
endif

dep_egl = dependency(
  'egl',
  required: get_option('egl'),
)
if dep_egl.found()
  add_project_arguments(
    '-DGLEW_EGL',
    language: 'c',
  )
endif

user_args = []
if get_option('default_library') != 'static'
  add_project_arguments(
    '-DGLEW_BUILD',
    language: 'c',
  )
else
  add_project_arguments(
    '-DGLEW_STATIC',
    language: 'c',
  )
  user_args += '-DGLEW_STATIC'
endif

c_args = []
ld_args = []
glew_sources = files('src/glew.c')
if host_system == 'windows'
  windows = import('windows')

  glew_sources += windows.compile_resources('build/glew.rc')
  name = 'glew32'

  if cc.get_argument_syntax() == 'msvc'
    c_args += ['-DVC_EXTRALEAN', '-GS-']
    ld_args += ['-BASE:0x62AA0000', '-nodefaultlib', '-noentry']
  else
    c_args += ['-fno-builtin', '-fno-stack-protector']
    ld_args += '-nostdlib'
  endif
else
  name = 'GLEW'
  c_args = cc.get_supported_arguments(
    '-ansi',
    '-W',
    '-Wcast-qual',
    '-fno-stack-protector',
  )
endif

glew_include = include_directories('include')

# The windows version shoould not have a version set
libglew = library(
  name,
  glew_sources,
  c_args: c_args,
  link_args: ld_args,
  include_directories: glew_include,
  dependencies: [dep_gl, dep_egl, dep_glu],
  soversion: host_system == 'windows' ? '' : meson.project_version().split('.')[0],
  version: meson.project_version(),
  install: true,
)

install_headers(
  'include/GL/wglew.h',
  'include/GL/glew.h',
  'include/GL/glxew.h',
  subdir: 'GL',
)

glew_dep = declare_dependency(
  link_with: libglew,
  compile_args: user_args,
  include_directories: glew_include,
  dependencies: [dep_gl, dep_egl, dep_glu],
)
