(1) Download and install MSYS2 from "https://www.msys2.org/".

(2) This will install both MSYS2 and MinGW to the msys2 directory on your local disk. Go into the msys2 folder and
open a MinGW64 shell as administrator.    

(3) In the MinGW shell, type in the following commands to install the required packages using the Pacman package manager:

    pacman -Syu		(NOTE: You will have to perform this call and close the shell several times for it to complete!)
    pacman -S mingw-w64-x86_64-python-pip
    pacman -S mingw-w64-x86_64-python-matplotlib
    pacman -S mingw-w64-x86_64-gcc mingw-w64-x86_64-gtk3 mingw-w64-x86_64-python3 mingw-w64-x86_64-python3-gobject
    pacman -S git make nano pkg-config gcc

(4) Clone the GitHub repository with the example showing the error in question:

    mkdir testing
    cd testing
    git clone https://github.com/LeighKorbel/pyinstaller_msys2_issue.git
    cd pyinstaller_msys2_issue

(5) Download PyInstaller using Python's pip package manager:

    pip install pyinstaller

(6) Now we are ready to start compiling. The first thing we need to do is run PyInstaller on the python script. This
    script is just a simple matplotlib plot that is embedded within a GTK window. Run the following commands:

    cd gtk_simple_plot
    pyinstaller plot.py

(7) After PyInstaller finishes, an executable file will now be located in "/dist/plot". This can be run from the 
    command line by:

    ./dist/plot/plot.exe

(8) Now that we have verified that this works, we want to compile a very basic GTK application in C that only has a 
    button that will spawn the script using GLib's g_spawn() function. However, this needs to be done in a MSYS2 shell
    because MinGW does not have the system headers needed in it. So you need to open an MSYS2 shell now as an administrator.
    After this is done, we will need to set an environment variable for pkg_config, which is used by GTK. I would
    recommend adding the following line to the .bashrc file and then closing and reopening the MSYS2 shell, but it is
    not required (you will just need to remember to export again in the case that you close and reopen the shell). Once
    the MSYS2 shell is open type in:

    nano .bashrc

(9) Scroll to the bottom of the file and type in the following:

    export PKG_CONFIG_PATH="/mingw64/lib/pkgconfig:$PKG_CONFIG_PATH"

(10) Now exit and save by pressing "Ctrl-x", then "y", then "enter". Close and reopen the shell. Now enter the following commands:

    cd testing/call_gtk_plot_from_C
    make

(11) The GTK program will now be compiled and can be executed by running:

    ./launch.exe

(12) Press the button on the UI to launch the PyInstaller compiled matplotlib script. The following error should
     appear in the MSYS2 shell:

    Traceback (most recent call last):
      File "plotter.py", line 6, in <module>
      File "<frozen importlib._bootstrap>", line 991, in _find_and_load
      File "<frozen importlib._bootstrap>", line 975, in _find_and_load_unlocked
      File "<frozen importlib._bootstrap>", line 671, in _load_unlocked
      File "PyInstaller/loader/pyimod03_importers.py", line 531, in exec_module
      File "matplotlib/__init__.py", line 921, in <module>
      File "matplotlib/__init__.py", line 602, in matplotlib_fname
      File "matplotlib/__init__.py", line 599, in gen_candidates
      File "matplotlib/__init__.py", line 239, in wrapper
      File "matplotlib/__init__.py", line 502, in get_configdir
      File "matplotlib/__init__.py", line 444, in _get_xdg_config_dir
      File "pathlib.py", line 1104, in home
      File "pathlib.py", line 267, in gethomedir
    RuntimeError: Can't determine home directory

********************************************************************************************************************
QUESTION 1: WHAT IS THE CAUSE OF THIS ERROR? IS THIS FROM THE DUAL USE OF MinGW FOR PYINSTALLER AND MSYS2 FOR MAKE?
            IS IT A PATH ISSUE?
********************************************************************************************************************

(13) This error can be eliminated by performing the following: In the "call_gtk_plot_from_C" folder, edit both the 
     makefile and the launch.c code (in src/) to remove all instances of "../gtk_simple_plot/" and then copy and move 
     the plot.py script from the gtk_simple_plot directory into the call_gtk_plot_from_C directory. From a MinGW shell
     run make.

     NOW IT WILL MAKE EVERYTHING, EVEN THE C CODE.

(14) Run the program:

     ./launch.exe

     When you run this time it errors out with a gdk_pixbuf error. For some reason, in /dist/plot/lib/gdk-pixbuf there
     is no longer the folder "loaders" which does exist on the previous run of PyInstaller. Copy it from the dist/
     folder in gtk_simple_plot over to the dist/ folder in call_gtk_plot_from_C. Run the program again.

     NOW IT WORKS WITHOUT THE PREVIOUS HOME DIRECTORY ERROR.

********************************************************************************************************************
QUESTION 2: WHAT CAUSED IT TO WORK NOW? WAS IT THE FULL COMPILE IN MinGW? IS THERE A PATH THAT IS DIFFERENT NOW WITH
            THE PYTHON SCRIPT BEING MOVED INTO THE NEW DIRECTORY? WHY WAS THE PIXBUF MISSING?
********************************************************************************************************************

********************************************************************************************************************
QUESTION 3: WHY DOES THE C CODE COMPILE IN MinGW NOW? IT DOES NOT CONTAIN THE SYSTEM HEADERS REQUIRED FOR IT COMPILE?
********************************************************************************************************************