project('iputils', 'c',
	default_options : [
		'c_std=c99',
		'warning_level=2',
	],
	version : 's20180629') # keep in sync with: git describe | awk -F- '{print $1}'

cc = meson.get_compiler('c')

add_project_arguments(
	'-include', 'config.h',
	'-include', 'git-version.h',
	language : 'c'
)

conf = configuration_data()
conf.set_quoted('PACKAGE_NAME', meson.project_name())
conf.set('_GNU_SOURCE', 1, description : 'Enable GNU extensions on systems that have them.')

if cc.compiles('''
	#include <sys/types.h>
	#include <netinet/in.h>
	#include <netinet/icmp6.h>
	int main(void) {
		struct icmp6_nodeinfo nodeinfo;
		return 0;
	}
''', name : 'struct icmp6_nodeinfo')
	conf.set('HAVE_STRUCT_ICMP6_NODEINFO', 1,
		description : 'Defined if struct icmp6_nodeinfo exists.')
endif

if cc.compiles('''
	#include <sys/types.h>
	#include <sys/time.h>
	#include <time.h>
	int main(void) {
		struct tm time;
		return 0;
	}
''', name : 'struct tm time;')
	conf.set('TIME_WITH_SYS_TIME', 1,
		description : 'Defined if struct tm exists.')
endif

m_dep = cc.find_library('m')
resolv_dep = cc.find_library('resolv')
if cc.has_function('clock_gettime')
	rt_dep = cc.find_library('disabler-appears-to-disable-executable-build', required : false)
else
	rt_dep = cc.find_library('rt')
endif

opt = get_option('USE_CAP')
if opt == true
	cap_dep = cc.find_library('cap')
	conf.set('HAVE_LIBCAP', 1,
		description : 'Defined if libcap exists.')
else
	cap_dep = dependency('disabler-appears-to-disable-executable-build', required : false)
endif

opt = get_option('ARPING_DEFAULT_DEVICE')
if opt != ''
	conf.set_quoted('DEFAULT_DEVICE', opt, description : 'arping default device.')
endif

opt = get_option('USE_SYSFS')
if opt == true
	conf.set('USE_SYSFS', 1, description : 'If set use /sys file system.')
endif

opt = get_option('USE_IDN')
if opt == true
	idn_dep = cc.find_library('idn2', required : false)
	if idn_dep.found()
		add_project_arguments('-DUSE_IDN', language : 'c')
		conf.set('USE_IDN', 1,
			description : 'If set use Internationalized Domain Name library.')
	endif
else
	idn_dep = dependency('disabler-appears-to-disable-executable-build', required : false)
endif

opt = get_option('USE_CRYPTO')
if opt == 'nettle'
	crypto_dep = dependency('nettle')
	conf.set('USE_NETTLE', 1, description : 'If set use nettle crypto library.')
elif opt == 'gcrypt'
	crypto_dep = cc.find_library('gcrypt')
	conf.set('USE_GCRYPT', 1, description : 'If set use gcrypt crypto library.')
elif opt == 'openssl'
	crypto_dep = dependency('openssl')
	conf.set('USE_OPENSSL', 1, description : 'if set use openssl crypto library.')
elif opt == 'none'
	crypto_dep = dependency('disabler-appears-to-disable-executable-build', required : false)
	conf.set('PING6_NONCE_MEMORY', 1,
		description : 'If set RFC6744 random does not use any CRYPTO lib.')
endif

systemd = dependency('systemd', required : false)
systemdunitdir = get_option('systemdunitdir')
if systemdunitdir == '' and systemd.found()
	systemdunitdir = systemd.get_pkgconfig_variable('systemdsystemunitdir')
endif

build_arping = get_option('BUILD_ARPING')
build_clockdiff = get_option('BUILD_CLOCKDIFF')
build_ping = get_option('BUILD_PING')
build_rarpd = get_option('BUILD_RARPD')
build_rinfod = get_option('BUILD_RDISC')
build_tftpd = get_option('BUILD_TFTPD')
build_tracepath = get_option('BUILD_TRACEPATH')
build_traceroute6 = get_option('BUILD_TRACEROUTE6')

build_mans = get_option('BUILD_MANS')
build_html_mans = get_option('BUILD_HTML_MANS')

if build_rinfod == true
	opt = get_option('ENABLE_RDISC_SERVER')
	if opt == true
		conf.set('RDISC_SERVER', 1, description : 'If set build rdisc server.')
	endif
endif

build_ninfod = get_option('BUILD_NINFOD')
if build_ninfod == true
	if cc.has_header('stdio.h') and cc.has_header('stdlib.h') and cc.has_header('stddef.h')
		conf.set('STDC_HEADERS', 1, description : 'Defined if we have standard c headers.')
	endif
	foreach h : [
		'error.h',
		'gcrypt.h',
		'gnutls/openssl.h',
		'inttypes.h',
		'limits.h',
		'linux/rtnetlink.h',
		'memory.h',
		'netdb.h',
		'netinet/icmp6.h',
		'netinet/in.h',
		'netinet/ip6.h',
		'openssl/md5.h',
		'pwd.h',
		'string.h',
		'strings.h',
		'sys/capability.h',
		'syslog.h',
		'sys/time.h',
		'sys/types.h',
		'sys/uio.h',
		'sys/utsname.h',
		'unistd.h',
	]
		if cc.has_header(h)
			conf.set('HAVE_' + h.to_upper().underscorify(), 1,
				description : 'Define if ' + h + ' can be included.')
		endif
	endforeach
	threads = dependency('threads')
	if threads.found()
		conf.set('ENABLE_THREADS', 1,
			description : 'Defined if libpthread is found.')
	endif
	opt = get_option('NINFOD_MESSAGES')
	if opt == true
		conf.set('ENABLE_DEBUG', 1, description : 'Enable ninfod syslog messages.')
	endif
endif

git_version_h = vcs_tag(
	input : 'git-version.h.meson',
	output : 'git-version.h'
)

config_h = configure_file(
	output : 'config.h',
	configuration : conf)

setcap = find_program('setcap', '/usr/sbin/setcap', '/sbin/setcap', required : false)
if cap_dep.found() and setcap.found()
	perm_type = 'caps'
else
	perm_type = 'setuid'
endif

############################################################
if build_ping == true
	executable('ping', ['ping.c', 'ping_common.c', 'ping6_common.c', git_version_h],
		dependencies : [m_dep, cap_dep, idn_dep, crypto_dep, resolv_dep],
		install: true)
	meson.add_install_script('build-aux/setcap-setuid.sh',
		join_paths(get_option('prefix'), get_option('bindir')),
		'ping',
		perm_type,
		setcap.path()
	)
endif

if build_tracepath == true
	executable('tracepath', ['tracepath.c', git_version_h],
		dependencies : idn_dep,
		install: true)
endif

if build_traceroute6 == true
	executable('traceroute6', ['traceroute6.c', git_version_h],
		dependencies : [cap_dep, idn_dep],
		install: true)
	meson.add_install_script('build-aux/setcap-setuid.sh',
		join_paths(get_option('prefix'), get_option('bindir')),
		'traceroute6',
		perm_type,
		setcap.path()
	)
endif

if build_clockdiff == true
	executable('clockdiff', ['clockdiff.c', git_version_h],
		dependencies : [cap_dep],
		install: true)
	meson.add_install_script('build-aux/setcap-setuid.sh',
		join_paths(get_option('prefix'), get_option('bindir')),
		'clockdiff',
		perm_type,
		setcap.path()
	)
endif

if build_rinfod == true
	executable('rdisc', ['rdisc.c', git_version_h],
		install_dir: 'sbin',
		install: true)
	if systemd.found()
		subs = configuration_data()
		subs.set('sbindir', join_paths(get_option('prefix'), get_option('sbindir')))
		unit_file = configure_file(
			input: 'systemd/rdisc.service.in',
			output: 'rdisc@.service',
			configuration: subs
		)
		install_data(unit_file, install_dir: systemdunitdir)
	endif
endif

if build_arping == true
	executable('arping', ['arping.c', git_version_h],
		dependencies : [rt_dep, cap_dep, idn_dep],
		install: true)
	meson.add_install_script('build-aux/setcap-setuid.sh',
		join_paths(get_option('prefix'), get_option('bindir')),
		'arping',
		perm_type,
		setcap.path()
	)
endif

if build_tftpd == true
	executable('tftpd', ['tftpd.c', 'tftpsubs.c', git_version_h],
		install: true)
endif

if build_rarpd == true
	executable('rarpd', ['rarpd.c', git_version_h],
		install_dir: 'sbin',
		install: true)
	if systemd.found()
		subs = configuration_data()
		subs.set('sbindir', join_paths(get_option('prefix'), get_option('sbindir')))
		unit_file = configure_file(
			input: 'systemd/rarpd.service.in',
			output: 'rarpd@.service',
			configuration: subs
		)
		install_data(unit_file, install_dir: systemdunitdir)
	endif
endif

if build_ninfod == true
	if not crypto_dep.found()
		error('BUILD_NINFOD=true and USE_CRYPTO=none cannot be combined')
	endif
	subdir ('ninfod')
endif

if build_mans == true or build_html_mans == true
	subdir ('doc')
endif

############################################################
# FIXME: write tests
#test('ping to 127.0.0.1', p, args : ['-p 1', '127.0.0.1'])
