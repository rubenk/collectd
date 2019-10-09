project('collectd', 'c',
	version : '6.0',
	meson_version : '>= 0.48',
	default_options : [
		'buildtype=debugoptimized',
		'c_std=gnu99',
		'prefix=/opt/collectd',
	])

add_project_arguments('-DHAVE_CONFIG_H', language : 'c')
cc = meson.get_compiler('c')

sbindir = get_option('sbindir')

flex = find_program('flex')
bison = find_program('bison')

flexgen = generator(flex,
	output : '@PLAINNAME@.c',
	arguments : ['-o', '@OUTPUT@', '@INPUT@'])

bisongen = generator(bison,
	output : ['@BASENAME@.c', '@BASENAME@.h'],
	arguments : ['--output=@OUTPUT0@', '--defines=@OUTPUT1@', '@INPUT@'])

scanner = flexgen.process('src/liboconfig/scanner.l')
parser = bisongen.process('src/liboconfig/parser.y')

includes = include_directories('src', 'src/daemon')

libavltree = static_library('avltree',
	sources : [
		'src/utils/avltree/avltree.c',
		'src/utils/avltree/avltree.h'
	],
	include_directories : includes)

libcommon = static_library('common',
	sources : [
		'src/utils/common/common.c',
		'src/utils/common/common.h',
	],
	include_directories : includes)

libheap = static_library('heap',
	sources : [
		'src/utils/heap/heap.c',
		'src/utils/heap/heap.h',
	],
	include_directories : includes)

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
		includes])

libgcrypt = dependency('libgcrypt', version : '>=1.1')

libcollectdclient = static_library('collectdclient',
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
	dependencies : libgcrypt)

threads = dependency('threads')
libcap = dependency('libcap', required : false)
libdl = cc.find_library('dl')

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
	export_dynamic : true)

executable('collectdctl',
	sources : 'src/collectdctl.c',
	include_directories : [
		includes,
		'src/libcollectdclient',
	],
	link_with : libcollectdclient,
	install : true)

executable('collectdmon',
	sources : 'src/collectdmon.c',
	include_directories : includes,
	install : true,
	install_dir : sbindir)

executable('collectd-nagios',
	sources : 'src/collectd-nagios.c',
	include_directories : [
		includes,
		'src/libcollectdclient',
	],
	link_with : libcollectdclient,
	install : true)

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
	install : true)
