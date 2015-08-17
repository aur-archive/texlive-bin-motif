# Maintainer: Bernhard Walle <bernhard.walle@gmx.de>
# based on "texlive-bin" from francois at archlinux dot org
# AUR Category: office

pkgname=texlive-bin-motif
pkgver=2008
pkgrel=3
pkgdesc="TeX Live binaries, xdvi built with Motif instead of xaw"
license=('GPL')
arch=('i686' 'x86_64')
depends=('gcc-libs' 't1lib' 'gd' 'libsigsegv' 'openmotif')
optdepends=('psutils' 't1utils' 'perl')
makedepends=('clisp' 'ffcall' 'lzma-utils')
conflicts=('texlive-omega' 'texlive-bin')
options=('!makeflags' '!libtool')
url='http://tug.org/texlive/'
source=('texmf.cnf' 'ftp://tug.org/texlive/historic/2008/texlive-20080816-source.tar.lzma' \
	'ftp://ftp.archlinux.org/other/texlive/texlive-bin-2008-texmf.tar.lzma')
provides=('texlive-bin')
backup=(usr/share/texmf/web2c/texmf.cnf \
	usr/share/texmf-config/web2c/mktex.cnf \
	usr/share/texmf-config/web2c/updmap.cfg \
	usr/share/texmf-config/web2c/fmtutil.cfg \
	usr/share/texmf-config/tex/generic/config/language.dat \
	usr/share/texmf-config/tex/generic/config/pdftexconfig.tex \
	usr/share/texmf-config/ttf2pk/ttf2pk.cfg \
	usr/share/texmf-config/dvips/config/config.ps \
	usr/share/texmf-config/dvipdfm/dvipdfmx.cfg \
	usr/share/texmf-config/dvipdfm/config/config \
	usr/share/texmf-config/xdvi/XDvi)
md5sums=('4293b402c535002a65a3ecf62253d567'
         '554287c3e458da776edd684506048d45'
         'bac8aee05595fb80fcae8e864ba063f6')
build() {
   cd $startdir/src
   if [ "${CARCH}" = "x86_64" ]; then
     export CFLAGS="${CFLAGS} -fPIC"
     export CXXFLAGS="${CXXFLAGS} -fPIC"
   fi
   lzma --force -dc texlive-bin-2008-texmf.tar.lzma | tar xf - || return 1
   install -m755 -d $startdir/pkg/usr/share || return 1
   find texmf -type d -exec install -d -m755 $startdir/pkg/usr/share/'{}' \; || return 1
   find texmf -type f -exec install -m644 '{}' $startdir/pkg/usr/share/'{}' \; || return 1
   lzma --force -dc texlive-20080816-source.tar.lzma | tar xf - || return 1
   cd texlive-20080816-source || return 1
   # This trick is for avoiding exiting when latex is not available, 
   # since we do not build the xindy documentation anyway:
   sed -i~ 's|latex="no"|latex="yes"|' utils/xindy/configure || return 1
   test ! -d Work && mkdir Work
   cd Work
   echo "--> Here we go with configure..."
   ../configure --prefix=/usr \
	--datarootdir=$startdir/pkg/usr/share \
   	--datadir=$startdir/pkg/usr/share \
	--disable-multiplatform --without-dialog \
    --without-psutils --without-texinfo --without-t1utils \
	--with-system-zlib --with-system-pnglib \
	--with-system-ncurses --with-system-t1lib \
	--with-system-gd --with-fontconfig=/usr/lib \
	--with-system-freetype2 --with-freetype2-libdir=/usr/lib \
	--with-freetype2-include=/usr/include/freetype2 \
	--with-xdvi-x-toolkit=motif --with-cxx-runtime-hack \
    --without-omega --without-aleph --without-graphite || return 1
   ### FIXME these fixes should be temporary: send patch upstream 
   find utils/xindy -name Makefile -exec sed -i -e "s|^prefix =.\+$|prefix = $startdir/pkg/usr|" '{}' \; || return 1
   # we skip that, coz it requires an almost full texlive installation for compilation of the documentation:
   sed -i -e "s|^MAKE_RULES = make-rules|MAKE_RULES = ''|" -e "s|^DOCS = doc|DOCS = ''|" utils/xindy/Makefile || return 1
   #############################################################
   echo "-------------------------------------------------------"
   echo "--> ... and now we build the whole beast"
   make || return 1
   #############################################################
   echo "-------------------------------------------------------"
   echo "--> ... proceeding with make install"
   install -d -m755 $startdir/pkg/usr/share/man/man5
   make prefix=$startdir/pkg/usr texmf=$startdir/pkg/usr/share/texmf install || return 1
   echo "-------------------------------------------------------"
   echo "--> ...fixing wrong symlinks to scripts under /usr/bin/"
   for f in $startdir/pkg/usr/bin/* ; do
	   if [ -L $f ]; then
		   target=`ls -l "$f" | sed 's/^.\+ -> //'`
		   if [[ "$target" == ..* ]]; then
			   newtarget=`echo $target | sed -e 's|../|/usr/share/|'`
			   rm -f $f
			   ln -s $newtarget $f
			   test -f $startdir/pkg/$newtarget && chmod a+x $startdir/pkg/$newtarget
		   fi
	   fi
   done
   
 #############################################################
 ## CLEAN UP... 
   echo "Cleaning up"
   # remove tlmgr from PATH
   rm -f $startdir/pkg/usr/bin/tlmgr

   for d in $startdir/pkg/usr/texmf/scripts/*; do
	   dname=`basename $d`
	   test ! -d $startdir/pkg/usr/share/texmf/scripts/$dname && mv $d $startdir/pkg/usr/share/texmf/scripts/
   done
   rm -rf $startdir/pkg/usr/{texmf,texmf-dist}
   # most man files went to two different places:
   for i in 1 5; do
	   # remove pdf versions of manpages:
	   rm -f $startdir/pkg/usr/share/texmf/man/man$i/*.pdf
	   for f in $startdir/pkg/usr/share/texmf/doc/man/man$i/*; do
		   bf=`basename $f`
		   if [[ ! -f $startdir/pkg/usr/share/man/man$i/$bf ]]; then
			   mv -f $f $startdir/pkg/usr/share/man/man$i/
		   fi
	   done
   done
   # remove extra documentation:
   rm -rf $startdir/pkg/usr/share/texmf/doc/
   # those files are also in base, but "make install" duplicated them here:
   rm -rf $startdir/pkg/usr/share/texmf/bibtex/
   #TODO leave info files ?
   rm -rf $startdir/pkg/usr/share/{info,doc}
   # remove files that belong to texinfo
   for f in info infokey install-info makeinfo texi2dvi pdftexi2dvi texi2pdf texindex; do
	   rm -f $startdir/pkg/usr/share/man/man1/$f.1
   done
   for f in texinfo info ; do
	   rm -f $startdir/pkg/usr/share/man/man5/$f.5
   done
   # remove files that belong to t1utils
   for f in t1ascii t1asm t1binary t1disasm t1mac t1unmac; do
	   rm -f $startdir/pkg/usr/share/man/man1/$f.1
   done
   # remove files that belong to psutils
   for f in epsffit extractres fixdlsrps fixfmps fixmacps fixpsditps fixpspps fixscribeps fixtpps fixwfwps fixwpps fixwwps getafm includeres psbook psmerge psnup psresize psselect pstops; do
	   rm -f $startdir/pkg/usr/share/man/man1/$f.1
   done
   # remove man files that belong to omega/aleph
   for f in omega lambda odvicopy odvips odvitype ofm2opl opl2ofm otp2ocp outocp ovf2ovp ovp2ovf oxdvi ; do
	   rm -f $startdir/pkg/usr/share/man/man1/$f.1
   done
   # replace upstream texmf.cnf with ours
   rm -f $startdir/pkg/usr/share/texmf/web2c/texmf.cnf
   install -m644 $startdir/src/texmf.cnf $startdir/pkg/usr/share/texmf/web2c/texmf.cnf
   ## remove omega and aleph from fmtutil.cnf
   sed -i -e '/omega/d' -e '/aleph/d' $startdir/pkg/usr/share/texmf/web2c/fmtutil.cnf || return 1
   ###################################################################
   # copy config files to texmf-config tree
   install -d -m755 $startdir/pkg/usr/share/texmf-config/web2c
   install -d -m755 $startdir/pkg/usr/share/texmf-config/dvips/config
   install -d -m755 $startdir/pkg/usr/share/texmf-config/dvipdfm/config
   install -d -m755 $startdir/pkg/usr/share/texmf-config/dvipdfmx
   install -d -m755 $startdir/pkg/usr/share/texmf-config/tex/generic/config
   install -d -m755 $startdir/pkg/usr/share/texmf-config/xdvi
   cp -a $startdir/pkg/usr/share/texmf/web2c/mktex.cnf \
   	$startdir/pkg/usr/share/texmf-config/web2c/
   cp -a $startdir/pkg/usr/share/texmf/web2c/updmap.cfg \
   	$startdir/pkg/usr/share/texmf-config/web2c/
   cp -a $startdir/pkg/usr/share/texmf/web2c/fmtutil.cnf \
   	$startdir/pkg/usr/share/texmf-config/web2c/
   cp -a $startdir/pkg/usr/share/texmf/dvips/config/config.ps \
   	$startdir/pkg/usr/share/texmf-config/dvips/config/
   cp -a $startdir/pkg/usr/share/texmf/dvipdfm/config/config \
   	$startdir/pkg/usr/share/texmf-config/dvipdfm/config/
   cp -a $startdir/pkg/usr/share/texmf/dvipdfmx/dvipdfmx.cfg \
   	$startdir/pkg/usr/share/texmf-config/dvipdfmx/
   cp -a $startdir/pkg/usr/share/texmf/tex/generic/config/pdftexconfig.tex \
   	$startdir/pkg/usr/share/texmf-config/tex/generic/config/
   cp -a $startdir/pkg/usr/share/texmf/tex/generic/config/language.dat \
   	$startdir/pkg/usr/share/texmf-config/tex/generic/config/
   cp -a $startdir/pkg/usr/share/texmf/xdvi/XDvi \
   	$startdir/pkg/usr/share/texmf-config/xdvi/
   # clean updmap.cfg
   sed -i '/^\(Map\|MixedMap\)/d' $startdir/pkg/usr/share/texmf-config/web2c/updmap.cfg
   sed -i '/^#! \(Map\|MixedMap\)/d' $startdir/pkg/usr/share/texmf-config/web2c/updmap.cfg
   # fix hard-coded paths in /usr/bin/xindy
   sed -i "s|'/.\+/pkg/usr|'/usr|" $startdir/pkg/usr/bin/xindy
}


