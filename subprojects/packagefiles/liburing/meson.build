project(
  'liburing',
  ['c', 'cpp'],
  version: '2.5',
  license: '(MIT OR LGPL-2.1-only) AND (MIT OR GPL-2.0-only WITH Linux-syscall-note)',
  meson_version: '>=0.57.0',
  default_options: [
    'default_library=both',
    'c_std=c11',
    'cpp_std=c++11',
    'warning_level=2',
  ],
)

if host_machine.system() != 'linux'
  error('Everything except Linux is unsupported. uring is a Linux framework')
endif

project_version = meson.project_version().split('.')
major_version = project_version[0]
minor_version = project_version[1]

add_project_arguments(
  '-D_GNU_SOURCE',
  '-D__SANE_USERSPACE_TYPES__',
  '-include',
  meson.current_build_dir() + '/config-host.h',
  '-fno-stack-protector',
  '-Wno-unused-parameter',
  language: ['c', 'cpp'],
)

thread_dep = dependency('threads')

cc = meson.get_compiler('c')

has_stringop_overflow = cc.has_argument('-Wstringop-overflow')
has_array_bounds = cc.has_argument('-Warray-bounds')

has__kernel_rwf_t = cc.has_type(
  '__kernel_rwf_t',
  prefix: '#include <linux/fs.h>',
)

has__kernel_timespec = cc.has_members(
  'struct __kernel_timespec',
  'tv_sec',
  'tv_nsec',
  prefix: '#include <linux/time_types.h>',
)

code = '''#include <sys/types.h>
#include <fcntl.h>
#include <string.h>
#include <linux/openat2.h>
int main(int argc, char **argv)
{
  struct open_how how;
  how.flags = 0;
  how.mode = 0;
  how.resolve = 0;
  return 0;
}
'''
has_open_how = cc.compiles(
  code,
  name: 'open_how',
)

code = '''#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <linux/stat.h>
int main(int argc, char **argv)
{
  struct statx x;

  return memset(&x, 0, sizeof(x)) != NULL;
}
'''
has_statx = cc.compiles(
  code,
  name: 'statx',
)

code = '''#include <sys/types.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <linux/stat.h>
main(int argc, char **argv)
{
  struct statx x;

  return memset(&x, 0, sizeof(x)) != NULL;
}
'''
glibc_statx = cc.compiles(
  code,
  name: 'glibc_statx',
)

cpp = meson.get_compiler('cpp')

code = '''#include <iostream>
int main(int argc, char **argv)
{
  std::cout << "Test";
  return 0;
}
'''
has_cxx = cpp.compiles(
  code,
  name: 'C++',
)

has_ucontext = (cc.has_type(
  'ucontext_t',
  prefix: '#include <ucontext.h>',
)
and cc.has_function(
  'makecontext',
  prefix: '#include <ucontext.h>',
))

has_nvme_uring_cmd = cc.has_type(
  'struct nvme_uring_cmd',
  prefix: '#include <linux/nvme_ioctl.h>',
)

conf_data = configuration_data()
conf_data.set('CONFIG_HAVE_KERNEL_RWF_T', has__kernel_rwf_t)
conf_data.set('CONFIG_HAVE_KERNEL_TIMESPEC', has__kernel_timespec)
conf_data.set('CONFIG_HAVE_OPEN_HOW', has_open_how)
conf_data.set('CONFIG_HAVE_STATX', has_statx)
conf_data.set('CONFIG_HAVE_GLIBC_STATX', glibc_statx)
conf_data.set('CONFIG_HAVE_CXX', has_cxx)
conf_data.set('CONFIG_HAVE_UCONTEXT', has_ucontext)
conf_data.set('CONFIG_HAVE_NVME_URING', has_nvme_uring_cmd)
conf_data.set('CONFIG_HAVE_FANOTIFY', cc.has_header('sys/fanotify.h'))
configure_file(
  output: 'config-host.h',
  configuration: conf_data,
)

subdir('src')
subdir('man')

if get_option('examples')
  subdir('examples')
endif

if get_option('tests')
  if get_option('default_library') != 'both'
    error('default_library=both required to build tests')
  endif
  subdir('test')
endif

pkg_mod = import('pkgconfig')
pkg_mod.generate(
  libraries: liburing,
  name: 'liburing',
  version: meson.project_version(),
  description: 'io_uring library',
  url: 'http://git.kernel.dk/cgit/liburing/',
)

summary(
  {
    'bindir': get_option('bindir'),
    'libdir': get_option('libdir'),
    'datadir': get_option('datadir'),
  },
  section: 'Directories',
)
summary(
  {
    'examples': get_option('examples'),
    'tests': get_option('tests'),
  },
  section: 'Configuration',
  bool_yn: true,
)
