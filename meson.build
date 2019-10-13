# vim: set ft=meson ts=2 sw=2 sts=2 et

project('collectd', 'c',
  version : '5.8.0.1314.g3fb3915+',
  meson_version : '>= 0.52',
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
conf.set_quoted('PACKAGE_NAME', 'collectd')
conf.set('COLLECTD_USERAGENT', false, description : 'User agent for http requests')
conf.set10('COLLECT_DAEMON', true, description : 'whether or not to enable feature daemon mode')
conf.set10('COLLECT_DEBUG', false, description : 'whether or not to enable feature debugging')
conf.set10('COLLECT_GETIFADDRS', false, description : 'whether or not to enable feature getifaddrs under Linux')
conf.set10('COLLECT_WERROR', true, description : 'whether or not to enable feature building with -Werror')
conf.set('DATA_MAX_NAME_LEN', 128, description : 'Maximum length of data buffers')
conf.set10('FP_LAYOUT_NEED_NOTHING', true, description : 'Define if doubles are stored in x86 representation.')
conf.set10('NAN_STATIC_DEFAULT', true, description : 'Define if NAN is defined by default and can initialize static variables.')

os_aix = false
os_darwin = false
os_freebsd = false
os_linux = false
os_solaris = false

if host_machine.system() == 'aix'
  conf.set10('KERNEL_AIX', true, description : 'True if program is to be compiled for a AIX kernel')
  os_aix = true
elif host_machine.system() == 'darwin'
  conf.set10('KERNEL_DARWIN', true, description : 'True if program is to be compiled for a Darwin kernel')
  os_darwin = true
elif host_machine.system() == 'freebsd'
  conf.set10('KERNEL_FREEBSD', true, description : 'True if program is to be compiled for a FreeBSD kernel')
  os_freebsd = true
elif host_machine.system() == 'linux'
  conf.set10('KERNEL_LINUX', true, description : 'True if program is to be compiled for a Linux kernel')
  os_linux = true
elif host_machine.system() == 'sunos'
  conf.set10('KERNEL_SOLARIS', true, description : 'True if program is to be compiled for a Solaris kernel')
  os_solaris = true
endif

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

if os_darwin
  headers = [
    'IOKit/IOKitLib.h',
    'IOKit/IOTypes.h',
    'IOKit/ps/IOPowerSources.h',
    'IOKit/ps/IOPSKeys.h',
  ]
  foreach h : headers
    if cc.check_header(h)
      conf.set10('HAVE_' + h.underscorify().to_upper(), true,
        description : 'Define to 1 if you have the <@0@> header file.'.format(h)
      )
    endif
  endforeach
endif

sbindir = get_option('sbindir')
prefixdir = get_option('prefix')
plugindir = join_paths(prefixdir, get_option('libdir'), 'collectd')

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
  include_directories : includes,
)

libcommon = static_library('common',
  sources : [
    'src/utils/common/common.c',
    'src/utils/common/common.h',
  ],
  include_directories : includes,
)

libheap = static_library('heap',
  sources : [
    'src/utils/heap/heap.c',
    'src/utils/heap/heap.h',
  ],
  include_directories : includes,
)

libformat_graphite = static_library('format_graphite',
  sources : [
    'src/utils/format_graphite/format_graphite.c',
    'src/utils/format_graphite/format_graphite.h',
  ],
  include_directories : includes,
)

libformat_json = static_library('format_json',
  sources : [
    'src/utils/format_json/format_json.c',
    'src/utils/format_json/format_json.h',
  ],
  include_directories : includes,
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

lcc_ver = collectd_version.split('.')
lcc_major = lcc_ver[0].to_int()
lcc_minor = lcc_ver[1].to_int()
lcc_patch = lcc_ver[2].to_int()

lcc_string = '@0@.@1@.@2@'.format(lcc_major, lcc_minor, lcc_patch)

if lcc_ver.length() == 4
  lcc_extra = lcc_ver[3].to_int()
endif

if lcc_ver.length() == 5
  lcc_extra = '@0@.@1@'.format(lcc_ver[3], lcc_ver[4])
endif

lcc_features = configuration_data()
lcc_features.set('LCC_VERSION_MAJOR', lcc_major)
lcc_features.set('LCC_VERSION_MINOR', lcc_minor)
lcc_features.set('LCC_VERSION_PATCH', lcc_patch)
lcc_features.set('LCC_VERSION_EXTRA', lcc_extra)
lcc_features.set('LCC_VERSION_STRING', lcc_string)
lcc_features_h = configure_file(configuration : lcc_features,
                                input : 'src/libcollectdclient/collectd/lcc_features.h.in',
                                output : '@BASENAME@'
)

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
  'src/libcollectdclient/collectd/network_buffer.h',
  'src/libcollectdclient/collectd/network.h',
  'src/libcollectdclient/collectd/network_parse.h',
  'src/libcollectdclient/collectd/server.h',
  'src/libcollectdclient/collectd/types.h',
  lcc_features_h,
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
  export_dynamic : true,
  c_args : '-DPLUGINDIR="@0@"'.format(plugindir),
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

plugins = [
  'aggregation',
  'amqp',
  'amqp1',
  'apache',
  'apcups',
  'apple_sensors',
  'aquaero',
  'ascent',
  'barometer',
  'battery',
  'bind',
  'ceph',
  'cgroups',
  'check_uptime',
  'chrony',
  'connectivity',
  'conntrack',
  'contextswitch',
  'cpu',
  'cpufreq',
  'cpusleep',
  'curl',
  'curl_json',
  'curl_xml',
  'dbi',
  'df',
  'disk',
  'dns',
  'dpdkevents',
  'dpdkstat',
  'drbd',
  'email',
  'entropy',
  'ethstat',
  'exec',
  'fhcount',
  'filecount',
  'fscache',
  'gmond',
  'gps',
  'grpc',
  'hddtemp',
  'hugepages',
  'intel_pmu',
  'intel_rdt',
  'interface',
  'ipc',
  'ipmi',
  'iptables',
  'ipvs',
  'irq',
  'java',
  'load',
  'lpar',
  'lua',
  'lvm',
  'madwifi',
  'match_empty_counter',
  'match_hashed',
  'match_regex',
  'match_timediff',
  'match_value',
  'mbmon',
  'mcelog',
  'md',
  'memcachec',
  'memcached',
  'memory',
  'mic',
  'modbus',
  'mqtt',
  'multimeter',
  'mysql',
  'netapp',
  'netlink',
  'nfs',
  'nginx',
  'notify_desktop',
  'notify_email',
  'notify_nagios',
  'ntpd',
  'numa',
  'nut',
  'olsrd',
  'onewire',
  'openldap',
  'openvpn',
  'oracle',
  'ovs_events',
  'ovs_stats',
  'pcie_errors',
  'perl',
  'pinba',
  'ping',
  'postgresql',
  'powerdns',
  'processes',
  'procevent',
  'protocols',
  'python',
  'redis',
  'routeros',
  'rrdcached',
  'rrdtool',
  'sensors',
  'serial',
  'sigrok',
  'smart',
  'snmp',
  'snmp_agent',
  'statsd',
  'swap',
  'sysevent',
  'table',
  'tail',
  'tail_csv',
  'tape',
  'target_notification',
  'target_replace',
  'target_scale',
  'target_set',
  'target_v5upgrade',
  'tcpconns',
  'teamspeak2',
  'ted',
  'thermal',
  'threshold',
  'tokyotyrant',
  'turbostat',
  'unixsock',
  'uptime',
  'users',
  'uuid',
  'varnish',
  'virt',
  'vmem',
  'vserver',
  'wireless',
  'write_graphite',
  'write_http',
  'write_kafka',
  'write_log',
  'write_mongodb',
  'write_prometheus',
  'write_redis',
  'write_riemann',
  'write_sensu',
  'write_stackdriver',
  'write_syslog',
  'write_tsdb',
  'xencpu',
  'xmms',
  'zfs_arc',
  'zone',
  'zookeeper',
]

collectd_conf_settings = {
  'PACKAGE_NAME': 'collectd',
  'localstatedir': join_paths(get_option('prefix'), get_option('localstatedir')),
  'libdir': join_paths(get_option('prefix'), get_option('libdir')),
  'prefix': get_option('prefix'),
  'DEFAULT_LOG_LEVEL': 'debug',
  'LOAD_PLUGIN_SYSLOG': '##',
  'LOAD_PLUGIN_LOGFILE': '',
  'LOAD_PLUGIN_LOG_LOGSTASH': '##',
  'LOAD_PLUGIN_CSV': '##',
  'LOAD_PLUGIN_NETWORK': '##',
  'LOAD_PLUGIN_RRDTOOL': '##',
}

plugin_defaults = configuration_data(collectd_conf_settings)

foreach plugin : plugins
  plugin_defaults.set('BUILD_PLUGIN_' + plugin.to_upper() + '_TRUE', '#')
  set_variable('plugin_' + plugin, false)
endforeach
if os_darwin
  plugin_battery = true
endif

if os_linux
  plugin_battery = true
  plugin_cgroups = true
  plugin_conntrack = true
  plugin_contextswitch = true
  plugin_cpu = true
  plugin_cpufreq = true
  plugin_disk = true
  plugin_drbd = true
  plugin_entropy = true
  plugin_fhcount = true
  plugin_fscache = true
  plugin_hugepages = true
  plugin_interface = true
  plugin_ipc = true
  plugin_irq = true
  plugin_load = true
  plugin_lvm = true
  plugin_mcelog = true
  plugin_memory = true
  plugin_nfs = true
  plugin_numa = true
  plugin_processes = true
  plugin_protocols = true
  plugin_serial = true
  plugin_swap = true
  plugin_synproxy = true
  plugin_tcpconns = true
  plugin_thermal = true
  plugin_uptime = true
  plugin_vmem = true
  plugin_vserver = true
  plugin_wireless = true
  plugin_zfs_arc = true
endif

configure_file(
  configuration : plugin_defaults,
  output : 'collectd.conf',
  input : 'src/collectd.conf.in',
  install : true,
  install_dir : get_option('sysconfdir'),
)

shared_module('aggregation', [
    'src/aggregation.c',
    'src/utils/lookup/vl_lookup.c',
    'src/utils/lookup/vl_lookup.h',
  ],
  include_directories : includes,
  install : true,
  name_prefix : '',
  install_dir : join_paths(get_option('libdir'), 'collectd'),
  gnu_symbol_visibility : 'hidden',
)

if plugin_battery
  shared_module('battery', [
      'src/battery.c',
      'src/battery_statefs.c',
    ],
    include_directories : includes,
    install : true,
    name_prefix : '',
    install_dir : join_paths(get_option('libdir'), 'collectd'),
    gnu_symbol_visibility : 'hidden',
  )
endif

shared_module('cpu', 'src/cpu.c',
  include_directories : includes,
  install : true,
  name_prefix : '',
  install_dir : join_paths(get_option('libdir'), 'collectd'),
  gnu_symbol_visibility : 'hidden',
)

if plugin_cpufreq
  shared_module('cpufreq', 'src/cpufreq.c',
    include_directories : includes,
    install : true,
    name_prefix : '',
    install_dir : join_paths(get_option('libdir'), 'collectd'),
    gnu_symbol_visibility : 'hidden',
  )
endif

if plugin_cpusleep
  shared_module('cpusleep', 'src/cpusleep.c',
    include_directories : includes,
    install : true,
    name_prefix : '',
    install_dir : join_paths(get_option('libdir'), 'collectd'),
    gnu_symbol_visibility : 'hidden',
  )
endif

shared_module('csv', 'src/csv.c',
  include_directories : includes,
  install : true,
  name_prefix : '',
  install_dir : join_paths(get_option('libdir'), 'collectd'),
  gnu_symbol_visibility : 'hidden',
)

shared_module('logfile', 'src/logfile.c',
  include_directories : includes,
  install : true,
  name_prefix : '',
  install_dir : join_paths(get_option('libdir'), 'collectd'),
  gnu_symbol_visibility : 'hidden',
)

shared_module('syslog', 'src/syslog.c',
  include_directories : includes,
  install : true,
  name_prefix : '',
  install_dir : join_paths(get_option('libdir'), 'collectd'),
  gnu_symbol_visibility : 'hidden',
)

shared_module('write_log', 'src/write_log.c',
  include_directories : includes,
  install : true,
  name_prefix : '',
  install_dir : join_paths(get_option('libdir'), 'collectd'),
  gnu_symbol_visibility : 'hidden',
  link_with : [
    libformat_graphite,
    libformat_json,
  ],
)

shared_module('write_syslog', 'src/write_syslog.c',
  include_directories : includes,
  install : true,
  name_prefix : '',
  install_dir : join_paths(get_option('libdir'), 'collectd'),
  gnu_symbol_visibility : 'hidden',
)
