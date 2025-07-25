project(
  'pcre2',
  'c',
  license: 'BSD-3-Clause WITH PCRE2-exception',
  meson_version: '>=0.49.0',
  version: '10.45',
)

pcre2_8_lib_version = '0.14.0'
pcre2_16_lib_version = '0.14.0'
pcre2_32_lib_version = '0.14.0'
pcre2_posix_lib_version = '3.0.6'

c_compiler = meson.get_compiler('c')

platform_is_unixlike = host_machine.system() in [
  'dragonfly',
  'freebsd',
  'netbsd',
  'openbsd',
  'haiku',
  'linux',
  'android',
  'darwin',
  'cygwin',
]

platform_supports_sealloc = host_machine.system() in ['linux', 'netbsd']

# pcre2.h
pcredata = configuration_data()
varr = meson.project_version().split('.')
pcredata.set('PCRE2_MAJOR', varr[0])
pcredata.set('PCRE2_MINOR', varr[1])
pcredata.set('PCRE2_PRERELEASE', '')
pcredata.set('PCRE2_DATE', '2025-02-05')

# config.h
cdata = configuration_data()
cdata.set('LINK_SIZE', '2')
cdata.set('HEAP_LIMIT', '20000000')
cdata.set('MATCH_LIMIT', '10000000')
cdata.set('MATCH_LIMIT_DEPTH', 'MATCH_LIMIT')
cdata.set('NEWLINE_DEFAULT', '2')  # default in config.h.generic
cdata.set('PARENS_NEST_LIMIT', '250')
cdata.set('PCRE2GREP_BUFSIZE', '20480')
cdata.set('PCRE2GREP_MAX_BUFSIZE', '1048576')
cdata.set('MAX_NAME_SIZE', '128')
cdata.set('MAX_NAME_COUNT', '10000')
cdata.set('MAX_VARLOOKBEHIND', '255')

if c_compiler.has_function_attribute('visibility:default')
  cdata.set('PCRE2_EXPORT', '__attribute__ ((visibility ("default")))')
else
  cdata.set('PCRE2_EXPORT', '')
endif

jit_deps = []
if not get_option('jit').disabled() and platform_is_unixlike
  jit_deps += dependency('threads')
  cdata.set10('SUPPORT_JIT', true)
endif

if not get_option('jit_sealloc').disabled()
  foreach f : ['mkostemp', 'memfd_create', 'secure_getenv']
    if c_compiler.has_function(
      f,
      prefix: '#include <stdlib.h>',
      args: ['-D_GNU_SOURCE'],
    )
      cdata.set10('HAVE_@0@'.format(f.to_upper()), true)
    endif
  endforeach

  if cdata.has('HAVE_MKOSTEMP') and cdata.has('HAVE_SECURE_GETENV')
    if platform_supports_sealloc
      cdata.set10('SLJIT_PROT_EXECUTABLE_ALLOCATOR', true)
    elif get_option('jit_sealloc').enabled()
      error('Your configuration is not supported')
    else
      message('jit_sealloc is not supported on this platform')
    endif
  endif
endif

sources = files(
  'src/pcre2_auto_possess.c',
  'src/pcre2_chkdint.c',
  'src/pcre2_compile.c',
  'src/pcre2_compile_class.c',
  'src/pcre2_config.c',
  'src/pcre2_context.c',
  'src/pcre2_convert.c',
  'src/pcre2_dfa_match.c',
  'src/pcre2_error.c',
  'src/pcre2_extuni.c',
  'src/pcre2_find_bracket.c',
  'src/pcre2_jit_compile.c',
  'src/pcre2_maketables.c',
  'src/pcre2_match.c',
  'src/pcre2_match_data.c',
  'src/pcre2_newline.c',
  'src/pcre2_ord2utf.c',
  'src/pcre2_pattern_info.c',
  'src/pcre2_script_run.c',
  'src/pcre2_serialize.c',
  'src/pcre2_string_utils.c',
  'src/pcre2_study.c',
  'src/pcre2_substitute.c',
  'src/pcre2_substring.c',
  'src/pcre2_tables.c',
  'src/pcre2_ucd.c',
  'src/pcre2_valid_utf.c',
  'src/pcre2_xclass.c',
)

includes = include_directories('.', 'src')

check_headers = [
  'sys/stat.h',
  'sys/types.h',
  'assert.h',
  'dirent.h',
  'windows.h',
  'unistd.h',
]

foreach h : check_headers
  if c_compiler.has_header(h)
    cdata.set10('HAVE_' + h.underscorify().to_upper(), true)
  endif
endforeach

foreach f : ['bcopy', 'memmove', 'realpath', 'strerror']
  if c_compiler.has_function(f)
    cdata.set10('HAVE_@0@'.format(f.to_upper()), true)
  endif
endforeach

if c_compiler.links(
  '''
  int main(int c, char *v[]) {
    if (c)
      __builtin_unreachable();
    return (int)(*v[0]);
  }
''',
  name: '__builtin_unreachable',
)
  cdata.set10('HAVE_BUILTIN_UNREACHABLE', true)
endif

if c_compiler.links(
  '''
  #include <stddef.h>
  int main(void) {
    int a, b;
    size_t res;
    __builtin_mul_overflow(a, b, &res);
  }
''',
  name: '__builtin_mul_overflow',
)
  cdata.set10('HAVE_BUILTIN_MUL_OVERFLOW', true)
endif

if c_compiler.links(
  '''
  int main(void) {
    __assume(1);
    return 0;
  }
''',
  name: '__assume',
)
  cdata.set10('HAVE_BUILTIN_ASSUME', true)
endif

if c_compiler.compiles(
  '''
  int main(void) {
    char buf[128] __attribute__((uninitialized));
    (void)buf;
    return 0;
  }
''',
  name: '__attribute__((uninitialized))',
)
  cdata.set10('HAVE_ATTRIBUTE_UNINITIALIZED', true)
endif

cdata.set10('SUPPORT_PCRE2_8', true)
cdata.set10('SUPPORT_PCRE2_16', true)
cdata.set10('SUPPORT_PCRE2_32', true)
cdata.set10('SUPPORT_UNICODE', true)

if get_option('default_library') == 'static'
  static_defs = ['-DPCRE2_STATIC']
  pcre2_posix_defs = []
else
  static_defs = []
  pcre2_posix_defs = ['-DPCRE2POSIX_SHARED']
endif

pcre2_h = configure_file(
  input: 'src/pcre2.h.in',
  output: 'pcre2.h',
  configuration: pcredata,
)

chartables = configure_file(
  input: 'src/pcre2_chartables.c.dist',
  output: 'pcre2_chartables.c',
  copy: true,
)
sources += chartables

config_h = configure_file(
  output: 'config.h',
  configuration: cdata,
)

general_c_args = [static_defs, '-DHAVE_CONFIG_H', '-D_GNU_SOURCE']
if host_machine.system() == 'windows'
  general_c_args += ['-D_CRT_SECURE_NO_DEPRECATE', '-D_CRT_SECURE_NO_WARNINGS']
endif

pcre2_8_lib = library(
  'pcre2-8',
  sources,
  dependencies: jit_deps,
  include_directories: includes,
  c_args: [general_c_args, '-DPCRE2_CODE_UNIT_WIDTH=8'],
  version: pcre2_8_lib_version,
  install: true,
  gnu_symbol_visibility: 'hidden',
)

libpcre2_8 = declare_dependency(
  link_with: pcre2_8_lib,
  include_directories: includes,
  compile_args: static_defs,
)

pcre2_posix_lib = library(
  'pcre2-posix',
  'src/pcre2posix.c',
  dependencies: libpcre2_8,
  c_args: [general_c_args, '-DPCRE2_CODE_UNIT_WIDTH=8', pcre2_posix_defs],
  version: pcre2_posix_lib_version,
  install: true,
  gnu_symbol_visibility: 'hidden',
)

libpcre2_posix = declare_dependency(
  link_with: pcre2_posix_lib,
  include_directories: includes,
  compile_args: [static_defs, pcre2_posix_defs],
)

pcre2_16_lib = library(
  'pcre2-16',
  sources,
  dependencies: jit_deps,
  include_directories: includes,
  c_args: [general_c_args, '-DPCRE2_CODE_UNIT_WIDTH=16'],
  version: pcre2_16_lib_version,
  install: true,
  gnu_symbol_visibility: 'hidden',
)

libpcre2_16 = declare_dependency(
  link_with: pcre2_16_lib,
  include_directories: includes,
  compile_args: static_defs,
)

pcre2_32_lib = library(
  'pcre2-32',
  sources,
  dependencies: jit_deps,
  include_directories: includes,
  c_args: [general_c_args, '-DPCRE2_CODE_UNIT_WIDTH=32'],
  version: pcre2_32_lib_version,
  install: true,
  gnu_symbol_visibility: 'hidden',
)

libpcre2_32 = declare_dependency(
  link_with: pcre2_32_lib,
  include_directories: includes,
  compile_args: static_defs,
)

if get_option('grep')
  pcre2grep = executable(
    'pcre2grep',
    'src/pcre2grep.c',
    dependencies: libpcre2_8,
    c_args: general_c_args,
    install: true,
  )
endif

install_headers(pcre2_h, 'src/pcre2posix.h')

########### pkg-config #############

pkg = import('pkgconfig')

pkg.generate(
  pcre2_8_lib,
  name: 'libpcre2-8',
  description: 'PCRE2 - Perl compatible regular expressions C library (2nd API) with 8 bit character support',
  version: meson.project_version(),
)

pkg.generate(
  pcre2_16_lib,
  name: 'libpcre2-16',
  description: 'PCRE2 - Perl compatible regular expressions C library (2nd API) with 16 bit character support',
  version: meson.project_version(),
)

pkg.generate(
  pcre2_32_lib,
  name: 'libpcre2-32',
  description: 'PCRE2 - Perl compatible regular expressions C library (2nd API) with 32 bit character support',
  version: meson.project_version(),
)

#### tests

if (get_option('test')
and not meson.is_cross_build()  # wine wrappers are not bat-friendly, this would need better testing
)
  link_args = []
  if c_compiler.get_argument_syntax() == 'msvc'
    link_args += '/STACK:2500000'
  endif

  pcre2test = executable(
    'pcre2test',
    'src/pcre2test.c',
    dependencies: [libpcre2_8, libpcre2_posix, libpcre2_16, libpcre2_32],
    c_args: general_c_args,
    link_args: link_args,
  )

  pcre2posix_test = executable(
    'pcre2posix_test',
    'src/pcre2posix_test.c',
    dependencies: [libpcre2_8, libpcre2_posix],
    link_args: link_args,
  )
  test('pcre2posix_test', pcre2posix_test)

  if c_compiler.get_argument_syntax() == 'msvc'
    runtest = find_program('RunTest.bat')
    rungreptest = find_program('RunGrepTest.bat')
  else
    runtest = find_program('RunTest')
    rungreptest = find_program('RunGrepTest')
  endif

  test(
    'RunTest',
    runtest,
    should_fail: host_machine.system() == 'windows' and c_compiler.get_argument_syntax() != 'msvc',
    env: 'srcdir=@0@'.format(meson.current_source_dir()),
    workdir: meson.current_build_dir(),
  )

  if get_option('grep')
    test(
      'RunGrepTest',
      rungreptest,
      should_fail: host_machine.system() == 'windows' and c_compiler.get_argument_syntax() != 'msvc',
      env: 'srcdir=@0@'.format(meson.current_source_dir()),
      workdir: meson.current_build_dir(),
    )
  endif
endif
