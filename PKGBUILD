# Maintainer: Victrid <weihau.chiang at gmail dot com>
# Modified from Batuhan Baserdem <lastname dot firstname at gmail> https://github.com/bloatmode/matlab-aur

# Matlab packaging for archlinux

name=MATLAB

pkgbase=matlab
pkgname=('python-matlabengine' 'matlab')
pkgrel=4
# No need to modify the pkgver here, which will be determined by the script
# in the offline installer you provided.
pkgver=9.10.0.1710957
pkgdesc='A high-level language for numerical computation and visualization'
arch=('x86_64')
url='http://www.mathworks.com'
license=(custom)
makedepends=('gendesk' 'python' 'findutils' 'icoutils')
depends=(
  'ca-certificates'
  'lsb-release'
  'alsa-lib'
  'atk'
  'libcap'
  'libcups'
  'libdbus'
  'fontconfig'
  'libgcrypt'
  'gdk-pixbuf2'
  'gst-plugins-base'
  'gstreamer'
  'gtk2'
  'krb5'
  'nspr'
  'nss'
  'pam'
  'pango'
  'cairo'
  'libsm'
  'libsndfile'
  'libx11'
  'libxcb'
  'libxcomposite'
  'libxcursor'
  'libxdamage'
  'libxext'
  'libxfixes'
  'libxft'
  'libxi'
  'libxmu'
  'libxrandr'
  'libxrender'
  'libxslt'
  'libxss'
  'libxt'
  'libxtst'
  'libxxf86vm'
  'procps-ng'
  'xorg-server-xvfb'
  'x11vnc'
  'sudo'
  'zlib')
# These I got from arch before and afraid to play around.
# GCC: https://www.mathworks.com/support/requirements/supported-compilers.html
depends+=(
  'gconf'
  'glu'
  'gstreamer'
  'libunwind'
  'libxp'
  'libxpm'
  'portaudio'
  'qt5-svg'
  'qt5-webkit'
  'qt5-websockets'
  'qt5-x11extras'
  'xerces-c')
provides=('matlab-licenses' 'python-matlabengine' 'matlab-bin')
source=(
  "matlab.tar.zst" # Modify this to fit your file
  "matlab.fik"
  "matlab.lic"
  "matlab.script")
md5sums=(SKIP SKIP SKIP 'b6651d0199305f18ab8c0c464b86a9c7')

# Limit products to lower size, set this to true to do a partial install
partialinstall=false
# Example list of products for a partial install; check README.md for details
products=(
  "MATLAB"
  #---MATLAB Product Family---#
  "Curve_Fitting_Toolbox"           # Math and Optimization
  "Database_Toolbox"              # Database Access and Reporting
  "Deep_Learning_HDL_Toolbox"
  "Deep_Learning_Toolbox"
  "DSP_System_Toolbox"
  "Global_Optimization_Toolbox"
  "GPU_Coder"
  "MATLAB_Coder"                # Code Generation
  "MATLAB_Compiler"               # Application Deployement
  "MATLAB_Compiler_SDK"
  "Optimization_Toolbox"
  "Parallel_Computing_Toolbox"        # Parallel computing
  "Partial_Differential_Equation_Toolbox"
  "Reinforcement_Learning_Toolbox"
  "Statistics_and_Machine_Learning_Toolbox"   # AI, Data Science, Statistics
  "Symbolic_Math_Toolbox"
  "Text_Analytics_Toolbox"
  #---Application Products---#
  "Audio_Toolbox"
  "Bioinformatics_Toolbox"          # Computational Biology
  "Computer_Vision_Toolbox"
  "Image_Processing_Toolbox"          # Image Processing and Computer Vision
  "Signal_Processing_Toolbox"         # Signal Processing
  "Wavelet_Toolbox"
)

instdir="/usr/lib/${pkgbase}"

pkgver() {
  grep "<version>" "${srcdir}/${pkgbase}/VersionInfo.xml" | sed "s|\s*<version>\(.*\)</version>\s*|\1|g"
}

prepare() {
  # Extract file installation key
  release=$(grep "<release>" "${srcdir}/${pkgbase}/VersionInfo.xml" | sed "s|\s*<release>\(.*\)</release>\s*|\1|g")
  
  msg2 "Release from tarball: ${release}"
  
  _fik=$(grep -o '[0-9-]*' "${srcdir}/${pkgbase}.fik")
  
  msg2 "Modifying the installer settings..."
  
  _set="${srcdir}/${pkgbase}/installer_input.txt"
  sed -i "s|^# destinationFolder=|destinationFolder=${srcdir}/build|" "${_set}"
  sed -i "s|^# fileInstallationKey=|fileInstallationKey=${_fik}|"   "${_set}"
  sed -i "s|^# agreeToLicense=|agreeToLicense=yes|"           "${_set}"
  sed -i "s|^# licensePath=|licensePath=${srcdir}/matlab.lic|"    "${_set}"

  # Select products if partialinstall is set
  if [ "${partialinstall}" = 'true' ]; then
    for _prod in "${products[@]}"; do
      sed -i 's|^#\(product.'"${_prod}"'\)|\1|' "${_set}"
    done
  fi

  # Desktop file
  # Implemented a fix for intel gpus with mesa 20;
  # https://wiki.archlinux.org/index.php/MATLAB#OpenGL_acceleration
  # https://wiki.archlinux.org/index.php/Intel_graphics#Old_OpenGL_Driver_(i965)
  
  msg2 "Generating desktop file..."
  
  gendesk -f -n \
    --pkgname "${pkgbase}" \
    --pkgdesc "${pkgdesc}" \
    --name "MATLAB" \
    --categories "Development;Education;Science;Mathematics;IDE" \
    --mimetypes "application/x-matlab-data;text/x-matlab" \
    --icon "${pkgbase}" \
    --exec 'sh -c '\''if [ "${MATLAB_INTEL_OVERRIDE}" = "yes" ] ; then exec env MESA_LOADER_DRIVER_OVERRIDE=i965 GTK_PATH=/usr/lib/gtk-2.0 matlab -desktop ; else exec env GTK_PATH=/usr/lib/gtk-2.0 matlab -desktop ; fi'\'
    
}

build() {
  msg2 "Installing with original installer..."
  # Using the installer with the -inputFile parameter will automatically
  #   cause the installation to be non-interactive
  "${srcdir}/${pkgbase}/install" -inputFile "${srcdir}/${pkgbase}/installer_input.txt"

  msg2 "Build the python API"
  
  cd "${srcdir}/build/extern/engines/python"

  msg2 "Checking supported vs existing matlab versions"
  _matminor="$(find "${srcdir}/build/extern/engines/python" \
    -name 'matlabengineforpython3*.so' |
    sort |
    sed 's|.*matlabengineforpython3_\([0-9]\+\)\.so|\1|g' |
    tail -1)"
  _pytminor="$(python -c 'import sys; print(sys.version_info.minor)')"

  msg2 "Spoof version compatibility if not applicable"
  if [[ "${_pytminor}" != "${_matminor}" ]]; then
    _matcustom="${srcdir}/sitecustomize.py"
    touch "${_matcustom}"
    echo 'import sys'                               >> "${_matcustom}"
    echo "sys.version_info = (3, ${_matminor}, 0)"  >> "${_matcustom}"
  fi
  PYTHONPATH="${srcdir}" python setup.py build
  
  msg2 removing build licenses...
  rm -rf "${srcdir}/build/licenses/*"
}


package_python-matlabengine() {
  depends+=("python" "matlab")
  
  msg2 "Copying license"
  install -D -m644 "${srcdir}/${pkgbase}/license_agreement.txt" \
    "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
    
  msg2 "Package the python API"
  cd "${srcdir}/build/extern/engines/python"
  PYTHONPATH="${srcdir}" python setup.py install --root="${pkgdir}" --optimize 1 --skip-build

  msg2 "Spoofing trick to fool matlab into believing current python version is supported"
  _matminor="$(find "${srcdir}/build/extern/engines/python" \
    -name 'matlabengineforpython3*.so' |
    sort |
    sed 's|.*matlabengineforpython3_\([0-9]\+\)\.so|\1|g' |
    tail -1)"
  _prefix="$(python -c 'import sys; print(sys.prefix)')"
  _pytminor="$(python -c 'import sys; print(sys.version_info.minor)')"
  
  msg2 "Change around locations if spoofing is needed"
  if [[ "${_pytminor}" != "${_matminor}" ]]; then
    mv "${pkgdir}/${_prefix}/lib/python3".{"${_matminor}","${_pytminor}"}
    _egginfo="$(ls "${pkgdir}/${_prefix}/lib/python3.${_pytminor}/site-packages/"*"-py3.${_matminor}.egg-info")"
    mv "${_egginfo}" "${_egginfo%py3."${_matminor}".egg-info}py3.${_pytminor}.egg-info"
    sed -i "s|sys.version_info|(3, $_matminor, 0)|" \
      "${pkgdir}/${_prefix}/lib/python3.${_pytminor}/site-packages/matlab/engine/__init__.py"
  fi

  msg2 "Fix erronous referances in the _arch.txt files"
  errstr=$(realpath "${srcdir}/build/extern/engines/python/")
  trustr="${instdir}/extern/engines/python/"
  for _dir in \
    "${srcdir}/build/extern/engines/python/build/lib/matlab/engine" \
    "${pkgdir}/${_prefix}/lib/python3.${_pytminor}/site-packages/matlab/engine" \
    ; do
    sed -i "s|${errstr}|${trustr}|" "${_dir}/_arch.txt"
  done
}

package_matlab() {
  # Compilers should be optional depends
  msg2 "Determining compiler versions..."
  if [ "$(vercmp ${pkgver} "9.10" )" -ge "0" ]; then
    optdepends+=('gcc9: For MEX support'
                'gcc8-fortran: For MEX support')
    gccexc="gcc-9"
    gppexc="g++-9"
    gfortranlib="gcc8-fortran"
    gfortranexc="gfortran-8"
  elif [ "$(vercmp ${pkgver} "9.9" )" -ge "0" ]; then
    optdepends+=('gcc8: For MEX support'
                'gcc8-fortran: For MEX support')
    gccexc="gcc-8"
    gppexc="g++-8"
    gfortranlib="gcc8-fortran"
    gfortranexc="gfortran-8"
  elif [ "$(vercmp ${pkgver} "9.4" )" -ge "0" ]; then
    optdepends+=('gcc6: For MEX support'
                'gcc6-fortran: For MEX support')
    gccexc="gcc-6"
    gppexc="g++-6"
    gfortranlib="gcc6-fortran"
    gfortranexc="gfortran-6"
  elif [ "$(vercmp ${pkgver} "9.1" )" -ge "0" ]; then
    optdepends+=('gcc49: For MEX support')
    gccexc="gcc-49"
    gppexc="g++-49"
    gfortranlib="gcc49"
    gfortranexc="gfortran-49"
  elif [ "$(vercmp ${pkgver} "8.2" )" -ge "0" ]; then
    optdepends+=('gcc47: For MEX support')
    gccexc="gcc-47"
    gppexc="g++-47"
    gfortranlib="gcc47"
    gfortranexc="gfortran-47"
  else
    msg2 "You need to install the GCC for MEX support yourself."
    msg2 "Visit here to determine your needed GCC version."
    msg2 "https://www.mathworks.com/support/requirements/previous-releases.html"
    msg2 "Create your own GCC package with name \"gcc-matlab\", and link these excutables to /usr/bin:"
    msg2 "gcc-matlab g++-matlab gfortran-matlab"
    gccexc="gcc-matlab"
    gppexc="g++-matlab"
    gfortranlib="gcc-matlab"
    gfortranexc="gfortran-matlab"
  fi
  
  msg2 "Moving files from build area"
  install -dm755 "${pkgdir}/usr/lib/"
  mv "${srcdir}/build" "${pkgdir}/${instdir}"

  msg2 "Copying license"
  install -D -m644 "${srcdir}/${pkgbase}/license_agreement.txt" \
    "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"

  msg2 "Symlink executables"
  install -d -m755 "${pkgdir}/usr/bin/"
  for _executable in deploytool matlab mbuild activate_matlab.sh; do
    ln -s "${instdir}/bin/${_executable}" "${pkgdir}/usr/bin/${_executable}"
  done
  # This would otherwise conflict with mixtex
  ln -s "${instdir}/bin/mex" "${pkgdir}/usr/bin/mex-${pkgbase}"
  # This would otherwise conflict with mathematica
  ln -s "${instdir}/bin/mcc" "${pkgdir}/usr/bin/mcc-${pkgbase}"
  # Allow external software to find MATLAB linter binary
  ln -s "${instdir}/bin/glnxa64/mlint" "${pkgdir}/usr/bin/mlint"

  msg2 "Install desktop files"
  install -D -m644 "${srcdir}/${pkgbase}.desktop" \
    "${pkgdir}/usr/share/applications/${pkgbase}.desktop"
  install -Dm644 "${srcdir}/${pkgbase}/bin/glnxa64/cef_resources/matlab_icon.png" "$pkgdir/usr/share/pixmaps/$pkgbase.png"

  msg2 "Link mex options to ancient libraries"
  sysdir="bin/glnxa64/mexopts"
  mkdir -p "${pkgdir}/${instdir}/backup/${sysdir}"
  cp "${pkgdir}/${instdir}/${sysdir}/gcc_glnxa64.xml" \
    "${pkgdir}/${instdir}/backup/${sysdir}/"
  sed -i "s/gcc/${gccexc}/g" "${pkgdir}/${instdir}/${sysdir}/gcc_glnxa64.xml"
  cp "${pkgdir}/${instdir}/${sysdir}/g++_glnxa64.xml" \
    "${pkgdir}/${instdir}/backup/${sysdir}/"
  sed -i "s/g++/${gppexc}/g" "${pkgdir}/${instdir}/${sysdir}/g++_glnxa64.xml"
  cp "${pkgdir}/${instdir}/${sysdir}/gfortran.xml" \
    "${pkgdir}/${instdir}/backup/${sysdir}/"
  sed -i "s/gfortran/${gfortranexc}/g" "${pkgdir}/${instdir}/${sysdir}/gfortran.xml"
  cp "${pkgdir}/${instdir}/${sysdir}/gfortran6.xml" \
    "${pkgdir}/${instdir}/backup/${sysdir}/"
  sed -i "s/gfortran/${gfortranexc}/g" "${pkgdir}/${instdir}/${sysdir}/gfortran6.xml"

  msg2 "Remove unused library files"
  # <MATLABROOT>/sys/os/glnxa64/README.libstdc++
  sysdir="sys/os/glnxa64"
  install -d -m755 "${pkgdir}/${instdir}/backup/${sysdir}"
  mv "${pkgdir}/${instdir}/${sysdir}/"{libstdc++.so.6.0.25,libstdc++.so.6} \
    "${pkgdir}/${instdir}/backup/${sysdir}/"
  mv "${pkgdir}/${instdir}/${sysdir}/libgcc_s.so.1" \
    "${pkgdir}/${instdir}/backup/${sysdir}/"
  mv "${pkgdir}/${instdir}/${sysdir}/"{libgfortran.so.5.0.0,libgfortran.so.5} \
    "${pkgdir}/${instdir}/backup/${sysdir}/"
  mv "${pkgdir}/${instdir}/${sysdir}/"{libquadmath.so.0.0.0,libquadmath.so.0} \
    "${pkgdir}/${instdir}/backup/${sysdir}/"

  # make sure MATLAB can find proper libraries libgfortran.so.3
  mkdir -p "${pkgdir}/${instdir}/backup/bin"
  cp "${pkgdir}/${instdir}/bin/matlab" "${pkgdir}/${instdir}/backup/bin"
  # The gcc dependency should be determined at runtime.
  sed -i "1s#^#if pacman -Q "${gfortranlib}' > /dev/null 2>&1 ; then \n export GCCVERSION=$(pacman -Q '${gfortranlib}" | awk '{print \$2}' | cut -d- -f1) \nfi\n\n#" "${pkgdir}/${instdir}/bin/matlab"
  sed -i "1s/^/# Check the optional GCC dependency.\n/" "${pkgdir}/${instdir}/bin/matlab"
  sed -i 's|LD_LIBRARY_PATH="`eval echo $LD_LIBRARY_PATH`"|if [ -n "${GCCVERSION}" ]; then \n LD_LIBRARY_PATH="`eval echo $LD_LIBRARY_PATH`:/usr/lib/gcc/x86_64-pc-linux-gnu/${GCCVERSION}"; \n else \n LD_LIBRARY_PATH="`eval echo $LD_LIBRARY_PATH`" \n fi \n|g' "${pkgdir}/${instdir}/bin/matlab"

  msg2 "Install the script file to make scripting easier with matlab"
  install -Dm 0755 "${srcdir}/matlab.script" "${pkgdir}/usr/bin/matscript"
}
