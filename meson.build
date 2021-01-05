project('minizip', 'c',
  version: '4.2.2',
)

cc = meson.get_compiler('c')

have_msvc = cc.get_id() == 'msvc'

headers = [
  'mz.h',
  'mz_crypt.h',
  'mz_os.h',
  'mz_strm.h',
  'mz_strm_buf.h',
  'mz_strm_mem.h',
  'mz_strm_os.h',
  'mz_strm_split.h',
  'mz_zip.h',
  'mz_zip_rw.h',
]

sources = [
  'mz_crypt.c',
  'mz_os.c',
  'mz_strm.c',
  'mz_strm_buf.c',
  'mz_strm_mem.c',
  'mz_strm_split.c',
  'mz_zip.c',
  'mz_zip_rw.c',
]

mz_config = configuration_data()
dirent_includes = ''
foreach h : ['dirent.h', 'sys/dirent.h']
  have_header = cc.has_header(h)
  mz_config.set('HAVE_' + h.underscorify().to_upper(), have_header)
  if have_header
    dirent_includes += '#include <' + h + '>\n'
  endif
endforeach
mz_config.set('HAVE_INTTYPES_H', cc.has_header('inttypes.h'))
mz_config.set('HAVE_STDINT_H', cc.has_header('stdint.h'))
mz_config.set('HAVE_PDIR', cc.has_type('DIR *', prefix: dirent_includes))
mz_config.set('HAVE_FSEEKO', cc.has_function('fseeko', prefix: '#include <stdio.h>'))
mz_config.set('HAVE_SYMLINK', cc.has_function('symlink', prefix: '#include <unistd.h>'))
mz_config.set('HAVE_READLINK', cc.has_function('readlink', prefix: '#include <unistd.h>'))

headers += configure_file(
  input: 'mz_config.h.cmakein',
  output: 'mz_config.h',
  configuration: mz_config,
  format: 'cmake@',
)

cdata = configuration_data()

if cc.has_type('off64_t', prefix: '#include <unistd.h>')
  cdata.set('__USE_LARGEFILE64', true)
  cdata.set('_LARGEFILE64_SOURCE', true)
endif

zlib_dep = dependency('zlib', required: get_option('zlib'))
if zlib_dep.found()
  headers += 'mz_strm_zlib.h'
  sources += 'mz_strm_zlib.c'
  cdata.set('HAVE_ZLIB', true)
  cdata.set('ZLIB_COMPAT', true)
endif

liblzma_dep = dependency('liblzma', required: get_option('lzma'))
if liblzma_dep.found()
  headers += 'mz_strm_lzma.h'
  sources += 'mz_strm_lzma.c'
  cdata.set('HAVE_LZMA', true)
endif

cdata.set('MZ_ZIP_NO_ENCRYPTION', true)

iconv_dep = []

if host_machine.system() == 'windows'
  sources += [
    'mz_os_win32.c',
    'mz_strm_os_win32.c',
  ]
  cdata.set('_CRT_SECURE_NO_DEPRECATE', true)
else
  sources += [
    'mz_os_posix.c',
    'mz_strm_os_posix.c',
  ]
  cdata.set('_BSD_SOURCE', true)
  cdata.set('_DEFAULT_SOURCE', true)
  if not cc.has_header_symbol('features.h', '_POSIX_C_SOURCE')
    cdata.set('_POSIX_C_SOURCE', '200809L')
  endif
  if host_machine.system() == 'linux'
    cdata.set('_GNU_SOURCE', true)
  elif host_machine.system() == 'darwin'
    cdata.set('_DARWIN_C_SOURCE', true)
  endif

  have_iconv = cc.has_function('iconv_open')
  if not have_iconv
    iconv_dep = dependency('libiconv', required: false)
    have_iconv = iconv_dep.found()

    if not have_iconv and cc.has_header_symbol('iconv.h', 'iconv_open')
      iconv_dep = cc.find_library('iconv')
      have_iconv = true
    endif

    if not have_iconv
      iconv_dep = dependency('libiconv')
      have_iconv = true
    endif
  endif
  cdata.set('HAVE_ICONV', have_iconv)
endif

install_headers(headers, subdir: 'minizip')

configure_file(
  output: 'config.h',
  configuration: cdata,
)
add_project_arguments(have_msvc ? '/FI' : '-include', 'config.h', language: 'c')

minizip = library('minizip', sources,
  include_directories: include_directories('.'),
  dependencies: [zlib_dep, liblzma_dep, iconv_dep],
  install: true,
)
minizip_dep = declare_dependency(
  include_directories: include_directories('.'),
  link_with: minizip,
)
meson.override_dependency('minizip', minizip_dep)

pkg = import('pkgconfig')
pkg.generate(minizip,
  description: 'Minizip zip file manipulation library',
  url: 'https://github.com/nmoinvaz/minizip',
  subdirs: ['minizip'],
)
