project(
	'wf-recorder',
    'c',
	'cpp',
	version: '0.2.1',
	license: 'MIT',
	meson_version: '>=0.47.0',
	default_options: [
		'cpp_std=c++11',
        'c_std=c11',
		'warning_level=2',
		'werror=false',
	],
)

conf_data = configuration_data()

conf_data.set('default_codec', get_option('default_codec'))
conf_data.set('default_framerate', get_option('default_framerate'))
conf_data.set('default_acodec', get_option('default_acodec'))
conf_data.set('default_sample_rate', get_option('default_sample_rate'))
conf_data.set('fallback_sample_fmt', get_option('fallback_sample_fmt'))

include_directories(['.'])

add_project_arguments(['-Wno-deprecated-declarations'], language: 'cpp')

project_sources = ['src/frame-writer.cpp', 'src/main.cpp', 'src/averr.c']

wayland_client = dependency('wayland-client')
wayland_protos = dependency('wayland-protocols')

pulse = dependency('libpulse-simple', required : get_option('pulse'))

if pulse.found()
    conf_data.set('HAVE_PULSE', true)
    project_sources += 'src/pulse.cpp'
endif

opencl = dependency('OpenCL', required : get_option('opencl'))

if opencl.found()
    conf_data.set('HAVE_OPENCL', true)
    project_sources += 'src/opencl.cpp'
endif

libavutil = dependency('libavutil')
libavcodec = dependency('libavcodec')
libavformat = dependency('libavformat')
libavdevice = dependency('libavdevice', required: false)
sws = dependency('libswscale')
swr = dependency('libswresample')
threads = dependency('threads')

conf_data.set('HAVE_LIBAVDEVICE', libavdevice.found())

configure_file(input: 'config.h.in',
               output: 'config.h',
               configuration: conf_data)

scdoc = dependency('scdoc', version: '>=1.9.2', native: true, required: get_option('man-pages'))

if scdoc.found()
    scdoc_bin=find_program(scdoc.get_pkgconfig_variable('scdoc'), native: true)

    sh = find_program('sh', native: true)

    mandir=get_option('mandir')

    filename ='manpage/wf-recorder.1.scd'

    project_name='wf-recorder'
    section='1'
    output= '@0@.@1@'.format(project_name, section)

    custom_target(
  	project_name,
	input: filename,
	output:output,
	command: [
	      sh, '-c', '@0@ < @INPUT@ > @1@'.format(scdoc_bin.path(), output)
	  ],
	install: true,
	install_dir: '@0@/man@1@'.format(mandir, section)
	)
endif

subdir('proto')

dependencies = [
    wayland_client, wayland_protos,
    libavutil, libavcodec, libavformat, libavdevice,
    wf_protos, sws, threads, pulse, swr, opencl
]

executable('wf-recorder', project_sources,
        dependencies: dependencies,
        install: true)
