import os
import sys
import time
import atexit

# TODO: [Add flag for generating launch.json and tasks.json] better in additional python setup script

# Start build time measurement
time_at_start = time.time()

main_env = Environment()

# Color definitions
colors = {}
colors["cyan"] = "\033[96m"
colors["purple"] = "\033[95m"
colors["blue"] = "\033[94m"
colors["green"] = "\033[92m"
colors["yellow"] = "\033[93m"
colors["red"] = "\033[91m"
colors["end"] = "\033[0m"

##############################################################################
# UTILITY FUNCTIONS                                                          #
##############################################################################

# TODO: Move functions into another file


def setup_color_output():

    # If the output is not a terminal, remove the colors
    if not sys.stdout.isatty():
        for key, value in colors.iteritems():
            colors[key] = ""

    compile_source_message = "%sCompiling %s==> %s$SOURCE%s" % \
        (colors["blue"], colors["purple"], colors["yellow"], colors["end"])

    compile_shared_source_message = "%sCompiling shared %s==> %s$SOURCE%s" % \
        (colors["blue"], colors["purple"], colors["yellow"], colors["end"])

    link_program_message = "%sLinking Program %s==> %s$TARGET%s" % \
        (colors["red"], colors["purple"], colors["yellow"], colors["end"])

    link_library_message = "%sLinking Static Library %s==> %s$TARGET%s" % \
        (colors["red"], colors["purple"], colors["yellow"], colors["end"])

    ranlib_library_message = "%sRanlib Library %s==> %s$TARGET%s" % \
        (colors["red"], colors["purple"], colors["yellow"], colors["end"])

    link_shared_library_message = "%sLinking Shared Library %s==> %s$TARGET%s" % \
        (colors["red"], colors["purple"], colors["yellow"], colors["end"])

    java_library_message = "%sCreating Java Archive %s==> %s$TARGET%s" % \
        (colors["red"], colors["purple"], colors["yellow"], colors["end"])

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


def glob_recursive_dir_list(dirs, file_ext=("cpp", "cxx", "cc", "C", "c++", "c"), use_variant_dir=False):
    found_files = list()
    for dirpath in dirs:
        found_files.append(glob_recursive(dir=dirpath,
                                          file_ext=file_ext,
                                          use_variant_dir=use_variant_dir))
    return found_files


def glob_recursive(dir="src", file_ext=(".*"), use_variant_dir=True):
    # For benchmarking
    time_glob_start = time.time()

    found_files = list()
    try:
        for dirpath, dirnames, filenames in os.walk(dir):
            # Replace src dir with variant dir, if requested
            if use_variant_dir:
                rel_dirpath = main_env["variant_dir"] + dirpath[len(dir)+1:]
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
        print(colors["red"], "Glob recursive failed.", colors["end"])


def detect_platform(env):

    if env["platform"] != "":
        env["sel_platform"] = env["platform"]
    else:
        # Missing "platform" argument, try to detect platform automatically
        if (
            sys.platform.startswith("linux")
            or sys.platform.startswith("dragonfly")
            or sys.platform.startswith("freebsd")
            or sys.platform.startswith("netbsd")
            or sys.platform.startswith("openbsd")
        ):
            env["sel_platform"] = "linuxbsd"
        elif sys.platform == "darwin":
            env["sel_platform"] = "osx"
        elif sys.platform == "win32":
            env["sel_platform"] = "windows"
        else:
            print(colors["red"] + "Could not detect platform automatically. Available platforms: windows, linuxbsd, osx\n"
                  "Please run SCons again and select a valid platform: platform=<string>" + colors["end"])

        if env["sel_platform"] != "":
            print("Automatically detected platform: " + env["sel_platform"])


def generate_version_header(filepath, version):
    from datetime import datetime
    version_split = version.split(".")
    build_time = datetime.now().strftime("%d/%m/%Y %H:%M:%S")
    lines = ["#pragma once\n",
             "\n",
             "//This header is automatically generated. Manual changes will be overwritten.\n",
             "\n",
             f"#define VERSION_MAJOR    {version_split[0]}\n",
             f"#define VERSION_MINOR    {version_split[1]}\n",
             f"#define VERSION_REVISION {version_split[2]}\n",
             "\n",
             f"#define VERSION_STR \"{version}\"\n",
             f"#define BUILD_TIME \"{build_time}\"\n"]
    with open(os.path.join(filepath, "version.hpp"), "w") as version_header_file:
        version_header_file.writelines(lines)
        print("Version info header file generated.")


def scons_finish():

    # Check failures and print complete message
    failures = GetBuildFailures()
    if len(failures) == 0:
        # Empty
        if main_env.GetOption("clean"):
            print(f'{colors["blue"]}---CLEANED---{colors["end"]}')
        else:
            print(f'{colors["green"]}---BUILD SUCCESSFUL---{colors["end"]}')
    else:
        print(f'{colors["red"]}---BUILD FAILED---{colors["end"]}')
        for bf in failures:
            print(
                f'{colors["red"]}{bf.node} failed: {bf.errstr}{colors["end"]}')

    # Print elapsed time
    elapsed_time_sec = round(time.time() - time_at_start, 3)
    time_ms = round((elapsed_time_sec % 1) * 1000)
    print(
        f'[Time elapsed: {time.strftime("%H:%M:%S", time.gmtime(elapsed_time_sec))}.{time_ms:03}]')


##############################################################################
# BUILD CONFIGURATION SECTION                                                #
##############################################################################

main_env["project_name"] = "project"
#Version (major, minor, revision/patch)
main_env["version"] = "0.0.0"

should_generate_version_header = True
version_header_path = "src"

install_dir_path = "install"
source_dir_path = "src"

include_path = [

]

# Directories containing static and dynamic libraries (debug/release versions)
libs_path_debug = [

]
libs_path_release = [

]

# Filenames of the libraries which are actually used
libs_debug = [

]
libs_release = [

]


source_file_extensions = ("cpp", "cxx", "cc", "C", "c++", "c")

##############################################################################
# MAIN SCRIPT                                                                #
##############################################################################

main_env["CPPPATH"] = include_path
main_env["INCLUDE"] = include_path

setup_color_output()

# Command line options
opts = Variables([], ARGUMENTS)
opts.Add(EnumVariable("target", "Compilation target", "release",
                      allowed_values=("debug", "release"), ignorecase=2))
opts.Add(BoolVariable("build_lib", "Build the library instead of the executable", 0))
opts.Add("install_dir", "Installation directory")
opts.Add("platform", "Target platform (windows, linuxbsd ord osx)", "")
opts.Update(main_env)

libs_path = libs_path_debug if main_env["target"] == "debug" else libs_path_release
libs = libs_debug if main_env["target"] == "debug" else libs_release
include_path.append(source_dir_path)

# Platform specific configuration
detect_platform(main_env)

SConscript("SCompilerConfig", ["main_env", "colors"])

platform_str = "Undefined"

if main_env["sel_platform"] == "windows":
    platform_str = "🌆 Windows"
    # Windows specific tasks/configuration
elif main_env["sel_platform"] == "linuxbsd":
    platform_str = "🐧 Linux/BSD"
    # Linux specific tasks/configuration
elif main_env["sel_platform"] == "osx":
    platform_str = "🍏 macOS"
    # MacOS specific tasks/configuration
print(colors["yellow"] + f"PLATFORM:{platform_str}" + colors["end"])


# Configure separate directory for build files
main_env.VariantDir(main_env["variant_dir"], "src", duplicate=0)

# Link main program or library
if main_env["build_lib"] == 1:
    prog_or_lib = main_env.SharedLibrary(
        main_env["project_name"],
        source=glob_recursive(source_dir_path, source_file_extensions),
        LIBS=libs,
        LIBPATH=libs_path)
else:
    prog_or_lib = main_env.Program(
        main_env["project_name"],
        source=glob_recursive(source_dir_path, source_file_extensions),
        LIBS=libs,
        LIBPATH=libs_path)

# Install all necessary files and the MAIN EXECUTABLE in one directory
install_files = glob_recursive_dir_list(
    dirs=libs_path, file_ext=("dll", "so"))
main_env.Install(install_dir_path, install_files)
main_env.Install(install_dir_path, prog_or_lib)

# Install header for library
if main_env["build_lib"] == 1:
    main_env.Install(install_dir_path + "/include",
                     "src/"+main_env["project_name"]+".hpp")

if should_generate_version_header:
    generate_version_header(version_header_path, main_env["version"])

# Measure build time/print info at the end
atexit.register(scons_finish)
