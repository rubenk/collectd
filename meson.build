# vim: set ft=meson ts=2 sw=2 sts=2 et

project('collectd', 'c',
  version : '6.0',
  meson_version : '>= 0.48',
  default_options : [
    'buildtype=debugoptimized',
    'c_std=gnu99',
    'prefix=/opt/collectd',
    'werror=true',
  ])

add_project_arguments('-DHAVE_CONFIG_H', language : 'c')
cc = meson.get_compiler('c')

conf = configuration_data()

collectd_version = meson.project_version()
conf.set_quoted('PACKAGE_VERSION', collectd_version)

# TODO: make these dynamic
conf.set('COLLECTD_USERAGENT', false, description : 'User agent for http requests')
conf.set10('COLLECT_DAEMON', true, description : 'whether or not to enable feature daemon mode')
conf.set10('COLLECT_DEBUG', false, description : 'whether or not to enable feature debugging')
conf.set10('COLLECT_GETIFADDRS', false, description : 'whether or not to enable feature getifaddrs under Linux')
conf.set10('COLLECT_WERROR', true, description : 'whether or not to enable feature building with -Werror')
conf.set('DATA_MAX_NAME_LEN', 128, description : 'Maximum length of data buffers')
conf.set10('FP_LAYOUT_NEED_NOTHING', true, description : 'Define if doubles are stored in x86 representation.')
conf.set10('NAN_STATIC_DEFAULT', true, description : 'Define if NAN is defined by default and can initialize static variables.')

headers = [
  'arpa/inet.h',
  'dirent.h',
  'endian.h',
  'fcntl.h',
  'fnmatch.h',
  'fs_info.h',
  'fshelp.h',
  'grp.h',
  'ifaddrs.h',
  'inttypes.h',
  'kstat.h',
  'kvm.h',
  'libgen.h',
  'locale.h',
  'mntent.h',
  'mnttab.h',
  'netdb.h',
  'paths.h',
  'poll.h',
  'pthread_np.h',
  'pwd.h',
  'regex.h',
  'net/if.h',
  'stdint.h',
  'sys/endian.h',
  'sys/fs_types.h',
  'sys/fstyp.h',
  'sys/ioctl.h',
  'sys/isa_defs.h',
  'sys/mntent.h',
  'sys/mnttab.h',
  'sys/param.h',
  'sys/resource.h',
  'sys/select.h',
  'sys/socket.h',
  'sys/stat.h',
  'sys/statfs.h',
  'sys/statvfs.h',
  'sys/sysctl.h',
  'sys/time.h',
  'sys/types.h',
  'sys/un.h',
  'sys/vfs.h',
  'sys/vfstab.h',
  'sys/vmmeter.h',
  'syslog.h',
  'unistd.h',
  'wordexp.h',
]

foreach h : headers
  if cc.check_header(h)
    conf.set10('HAVE_' + h.underscorify().to_upper(), true,
      description : 'Define to 1 if you have the <@0@> header file.'.format(h)
    )
  else
    conf.set('HAVE_' + h.underscorify().to_upper(), false,
      description : 'Define to 1 if you have the <@0@> header file.'.format(h)
    )
  endif
endforeach

functions = [
  'asprintf',
  'getifaddrs',
  'getloadavg',
  'getmntent',
  'getmntent_r',
  'getpwnam',
  'getpwnam_r',
  'getutent',
  'getutxent',
  'host_statistics',
  'if_indextoname',
  'processor_info',
  'setenv',
  'setgroups',
  'setlocale',
  'statfs',
  'statvfs',
  'strerror_r',
  'sysctl',
  'sysctlbyname',
  'syslog',
  'thread_info',
]

foreach f : functions
  if cc.has_function(f)
    conf.set10('HAVE_' + f.underscorify().to_upper(), true,
      description : 'Define to 1 if you have the `@0@\' function.'.format(f)
    )
  endif
endforeach

if cc.compiles('''
  #include <sys/types.h>
  int main(void) { return htonll(0); }
''', args : '-Werror')
  conf.set10('HAVE_HTONLL', true, description : 'Define if the function htonll exists.')
endif

sbindir = get_option('sbindir')

flex = find_program('flex')
bison = find_program('bison')

flexgen = generator(flex,
  output : '@BASENAME@.c',
  arguments : ['-o', '@OUTPUT@', '@INPUT@']
)

bisongen = generator(bison,
  output : ['@BASENAME@.c', '@BASENAME@.h'],
  arguments : ['--output=@OUTPUT0@', '--defines=@OUTPUT1@', '@INPUT@']
)

scanner = flexgen.process('src/liboconfig/scanner.l')
parser = bisongen.process('src/liboconfig/parser.y')

pod2man = find_program('pod2man')

includes = include_directories('src', 'src/daemon')

libavltree = static_library('avltree',
  sources : [
    'src/utils/avltree/avltree.c',
    'src/utils/avltree/avltree.h'
  ],
  include_directories : includes
)

libcommon = static_library('common',
  sources : [
    'src/utils/common/common.c',
    'src/utils/common/common.h',
  ],
  include_directories : includes
)

libheap = static_library('heap',
  sources : [
    'src/utils/heap/heap.c',
    'src/utils/heap/heap.h',
  ],
  include_directories : includes
)

liboconfig = static_library('oconfig',
  sources : [
    'src/liboconfig/oconfig.c',
    'src/liboconfig/oconfig.h',
    'src/liboconfig/aux_types.h',
    scanner,
    parser
  ],
  include_directories : [
    'src/liboconfig',
    includes,
  ]
)

libgcrypt = dependency('libgcrypt', version : '>=1.1')
if cc.has_header('gcrypt.h')
  conf.set10('HAVE_GCRYPT_H', true)
endif

libcollectdclient = shared_library('collectdclient',
  sources : [
    'src/libcollectdclient/client.c',
    'src/libcollectdclient/network.c',
    'src/libcollectdclient/network_buffer.c',
    'src/libcollectdclient/network_parse.c',
    'src/libcollectdclient/server.c',
    'src/libcollectdclient/collectd/stdendian.h',
  ],
  include_directories : [
    'src/libcollectdclient',
    includes,
  ],
  dependencies : libgcrypt,
  install : true,
  version: '1.1.0'
)

collectdclient_headers = [
  'src/libcollectdclient/collectd/client.h',
  'src/libcollectdclient/collectd/lcc_features.h',
  'src/libcollectdclient/collectd/network_buffer.h',
  'src/libcollectdclient/collectd/network.h',
  'src/libcollectdclient/collectd/network_parse.h',
  'src/libcollectdclient/collectd/server.h',
  'src/libcollectdclient/collectd/types.h',
]

install_headers(collectdclient_headers, subdir : 'collectd')

threads = dependency('threads')
libdl = cc.find_library('dl')

libcap = dependency('libcap', required : false)
if cc.has_function('cap_get_proc', dependencies : libcap)
  conf.set10('HAVE_CAPABILITY', true,
    description : 'Define to 1 if you have cap_get_proc() (-lcap).'
  )
endif

collectd_sources = [
  'src/daemon/collectd.c',
  'src/daemon/cmd.c',
  'src/daemon/cmd.h',
  'src/daemon/collectd.c',
  'src/daemon/collectd.h',
  'src/daemon/configfile.c',
  'src/daemon/configfile.h',
  'src/daemon/filter_chain.c',
  'src/daemon/filter_chain.h',
  'src/daemon/globals.c',
  'src/daemon/globals.h',
  'src/utils/metadata/meta_data.c',
  'src/utils/metadata/meta_data.h',
  'src/daemon/plugin.c',
  'src/daemon/plugin.h',
  'src/daemon/utils_cache.c',
  'src/daemon/utils_cache.h',
  'src/daemon/utils_complain.c',
  'src/daemon/utils_complain.h',
  'src/daemon/utils_llist.c',
  'src/daemon/utils_llist.h',
  'src/daemon/utils_random.c',
  'src/daemon/utils_random.h',
  'src/daemon/utils_subst.c',
  'src/daemon/utils_subst.h',
  'src/daemon/utils_time.c',
  'src/daemon/utils_time.h',
  'src/daemon/types_list.c',
  'src/daemon/types_list.h',
  'src/daemon/utils_threshold.c',
  'src/daemon/utils_threshold.h',
]

executable('collectd',
  sources : collectd_sources,
  include_directories : includes,
  dependencies : [
    threads,
    libcap,
    libdl,
  ],
  link_with : [
    libavltree,
    libcommon,
    libheap,
    liboconfig
  ],
  install : true,
  install_dir : sbindir,
  export_dynamic : true
)

rpath = get_option('prefix') + '/' + get_option('libdir')

executable('collectdctl',
  sources : 'src/collectdctl.c',
  include_directories : [
    includes,
    'src/libcollectdclient',
  ],
  link_with : libcollectdclient,
  install : true,
  install_rpath : rpath,
)

executable('collectdmon',
  sources : 'src/collectdmon.c',
  include_directories : includes,
  install : true,
  install_dir : sbindir
)

executable('collectd-nagios',
  sources : 'src/collectd-nagios.c',
  include_directories : [
    includes,
    'src/libcollectdclient',
  ],
  link_with : libcollectdclient,
  install : true,
  install_rpath : rpath,
)

executable('collectd-tg',
  sources : 'src/collectd-tg.c',
  include_directories : [
    includes,
    'src/libcollectdclient',
  ],
  link_with : [
    libcollectdclient,
    libheap,
  ],
  install : true,
  install_rpath : rpath,
)

configure_file(
  output : 'config.h',
  configuration : conf
)

manpages = [
  ['collectd', '1'],
  ['collectd-nagios', '1'],
  ['collectd-tg', '1'],
  ['collectdctl', '1'],
  ['collectdmon', '1'],
  ['collectd-email', '5'],
  ['collectd-exec', '5'],
  ['collectd-java', '5'],
  ['collectd-lua', '5'],
  ['collectd-perl', '5'],
  ['collectd-python', '5'],
  ['collectd-snmp', '5'],
  ['collectd-threshold', '5'],
  ['collectd-unixsock', '5'],
  ['collectd.conf', '5'],
  ['types.db', '5'],
]

foreach tuple : manpages
  stem = tuple[0]
  section = tuple[1]
  man = stem + '.' + section
  pod = join_paths('src', stem + '.pod')
  mandirn = join_paths(get_option('mandir'), 'man' + section)

  custom_target(man,
    output : man,
    input : pod,
    install : true,
    install_dir : join_paths(get_option('prefix'), get_option('mandir'), 'man' + section),
    build_by_default : true,
    command : [pod2man, '--section', section, '--release', collectd_version, '--center=collectd', '@INPUT@', '@OUTPUT@']
  )
endforeach

install_data([
  'src/types.db',
  'src/postgresql_default.conf'
])

pkg = import('pkgconfig')
pkg.generate(libraries : libcollectdclient,
             name : 'libcollectdclient',
             description : 'Client library for the unixsock plugin of collectd.'
)
