import os
import sys
import time
import atexit

# TODO: [Add flag for generating launch.json and tasks.json] better in additional python setup script

# Start build time measurement
time_at_start = time.time()

# Color definitions
colors = {}
colors['cyan'] = '\033[96m'
colors['purple'] = '\033[95m'
colors['blue'] = '\033[94m'
colors['green'] = '\033[92m'
colors['yellow'] = '\033[93m'
colors['red'] = '\033[91m'
colors['end'] = '\033[0m'

##############################################################################
# UTILITY FUNCTIONS                                                          #
##############################################################################


def setupColorOutput():

    # If the output is not a terminal, remove the colors
    if not sys.stdout.isatty():
        for key, value in colors.iteritems():
            colors[key] = ''

    compile_source_message = '%sCompiling %s==> %s$SOURCE%s' % \
        (colors['blue'], colors['purple'], colors['yellow'], colors['end'])

    compile_shared_source_message = '%sCompiling shared %s==> %s$SOURCE%s' % \
        (colors['blue'], colors['purple'], colors['yellow'], colors['end'])

    link_program_message = '%sLinking Program %s==> %s$TARGET%s' % \
        (colors['red'], colors['purple'], colors['yellow'], colors['end'])

    link_library_message = '%sLinking Static Library %s==> %s$TARGET%s' % \
        (colors['red'], colors['purple'], colors['yellow'], colors['end'])

    ranlib_library_message = '%sRanlib Library %s==> %s$TARGET%s' % \
        (colors['red'], colors['purple'], colors['yellow'], colors['end'])

    link_shared_library_message = '%sLinking Shared Library %s==> %s$TARGET%s' % \
        (colors['red'], colors['purple'], colors['yellow'], colors['end'])

    java_library_message = '%sCreating Java Archive %s==> %s$TARGET%s' % \
        (colors['red'], colors['purple'], colors['yellow'], colors['end'])

    main_env.Replace(CXXCOMSTR=compile_source_message)
    main_env.Replace(CCCOMSTR=compile_source_message)
    main_env.Replace(SHCCCOMSTR=compile_shared_source_message)
    main_env.Replace(SHCXXCOMSTR=compile_shared_source_message)
    main_env.Replace(ARCOMSTR=link_library_message)
    main_env.Replace(RANLIBCOMSTR=ranlib_library_message)
    main_env.Replace(SHLINKCOMSTR=link_shared_library_message)
    main_env.Replace(LINKCOMSTR=link_program_message)
    main_env.Replace(JARCOMSTR=java_library_message)
    main_env.Replace(JAVACCOMSTR=compile_source_message)


def GlobRecursiveDirList(dirs, file_ext=("cpp", "cxx", "cc", "C", "c++", "c"), use_variant_dir=False):
    found_files = list()
    for dirpath in dirs:
        found_files.append(GlobRecursive(
            dir=dirpath, file_ext=file_ext, use_variant_dir=use_variant_dir))
    return found_files


def GlobRecursive(dir="src", file_ext=("cpp", "cxx", "cc", "C", "c++", "c"), use_variant_dir=True):
    # For benchmarking
    time_glob_start = time.time()

    found_files = list()
    try:
        for dirpath, dirnames, filenames in os.walk(dir):
            # Replace src dir with variant dir, if requested
            if use_variant_dir:
                rel_dirpath = main_env['variant_dir'] + dirpath[len(dir)+1:]
            else:
                rel_dirpath = dirpath
            for filename in filenames:
                if filename.endswith(file_ext):
                    #print(os.path.join(rel_dirpath, filename))
                    found_files.append(os.path.join(rel_dirpath, filename))
        
        # For benchmarking
        glob_elapsed_time_ms = (time.time() - time_glob_start) * 1000
        print(
            f"GlobRecursive > [Files collected. Took {glob_elapsed_time_ms:.1f} ms]")
        return found_files
    except Exception as e:
        print(e)
        print(colors['red'], "Glob recursive failed.", colors['end'])


def scons_finish():
    # Check failures and print complete message
    # Check failures and print complete message
    failures = GetBuildFailures()
    if len(failures) == 0:
        # Empty
        if main_env.GetOption('clean'):
            print(f"{colors['blue']}>>>CLEANED<<<{colors['end']}")
        else:
            print(f"{colors['green']}>>>BUILD SUCCESSFUL<<<{colors['end']}")
    else:
        print(f"{colors['red']}>>>BUILD FAILED<<<{colors['end']}")
        for bf in failures:
            print(
                f"{colors['red']}{bf.node} failed: {bf.errstr}{colors['end']}")

    # Print elapsed time
    elapsed_time_sec = round(time.time() - time_at_start, 3)
    time_ms = round((elapsed_time_sec % 1) * 1000)
    print(
        f"[Time elapsed: {time.strftime('%H:%M:%S', time.gmtime(elapsed_time_sec))}.{time_ms:03}]")

##############################################################################
# MAIN SCRIPT                                                                #
##############################################################################


include_path = [

]

# Directories containing static and dynamic libraries (debug/release versions)
libs_path_debug = [

]
libs_path_release = [

]

# filenames of the libraries which are actually used
libs_debug = [

]
libs_release = [

]

# Begin build setup
main_env = Environment(CPPPATH=include_path, INCLUDE=include_path)

# Command line option for choise between debug/release targets
opts = Variables([], ARGUMENTS)
opts.Add(EnumVariable('target', 'Compilation target', 'release',
                      allowed_values=('debug', 'release'), ignorecase=2))
opts.Add(BoolVariable("build_lib", "Build the library instead of the executable", 0))
opts.Add("install_dir", "Installation directory")
opts.Update(main_env)

setupColorOutput()

SConscript("SCompilerConfig", ["main_env", "colors"])

libs_path = libs_path_debug if main_env['target'] == 'debug' else libs_path_release
libs = libs_debug if main_env['target'] == 'debug' else libs_release

# main_env.Tool('msvc')

main_env['project_name'] = "project"
main_env['version'] = "0.0.0"

# Platform specific configuration
platform_str = "UNKNOWN"
if main_env['PLATFORM'] == 'win32':
    platform_str = "🌆 Windows"
    # Windows specific tasks/configuration
elif main_env['PLATFORM'] == 'posix':
    platform_str = "🐧 Linux"
    # Linux specific tasks/configuration
elif main_env['PLATFORM'] == 'posix':
    platform_str = "🍏 MacOS"
    # MacOS specific tasks/configuration
print(colors['yellow'] + f"PLATFORM:{platform_str}" + colors['end'])

install_dir_path = "install"
#main_env["TARGET_ARCH"] = "amd64"

# Configure separate directory for build files
main_env.VariantDir(main_env['variant_dir'], "src", duplicate=0)

# Link main program or library
if main_env['build_lib'] == 1:
    prog_or_lib = main_env.SharedLibrary(
        main_env['project_name'],
        source=GlobRecursive(),
        LIBS=libs,
        LIBPATH=libs_path)
else:
    prog_or_lib = main_env.Program(
        main_env['project_name'],
        source=GlobRecursive(),
        LIBS=libs,
        LIBPATH=libs_path)

# Install all necessary files and the MAIN EXECUTABLE in one directory
install_files = GlobRecursiveDirList(
    dirs=libs_path_debug if main_env['target'] == 'debug' else libs_path_release, file_ext=("dll", "so"))
main_env.Install(install_dir_path, install_files)
main_env.Install(install_dir_path, prog_or_lib)

# Install header for library
if main_env['build_lib'] == 1:
    main_env.Install(install_dir_path + "/include",
                     "src/"+main_env['project_name']+".hpp")

# Measure build time/print info at the end
atexit.register(scons_finish)
