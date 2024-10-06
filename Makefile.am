#
#
#  OO_Copyright_BEGIN
#
#
#  Copyright 2010, 2020 IBM Corp. All rights reserved.
#
#  Redistribution and use in source and binary forms, with or without
#   modification, are permitted provided that the following conditions
#  are met:
#  1. Redistributions of source code must retain the above copyright
#     notice, this list of conditions and the following disclaimer.
#  2. Redistributions in binary form must reproduce the above copyright
#     notice, this list of conditions and the following disclaimer in the
#  documentation and/or other materials provided with the distribution.
#  3. Neither the name of the copyright holder nor the names of its
#     contributors may be used to endorse or promote products derived from
#     this software without specific prior written permission.
#
#  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS ``AS IS''
#  AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
#  IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
#  ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
#  LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
#  CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
#  SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
#  INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
#  CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
#  ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
#  POSSIBILITY OF SUCH DAMAGE.
#
#
#  OO_Copyright_END
#

ACLOCAL_AMFLAGS = -I m4

nobase_pkginclude_HEADERS = config.h

SUBDIRS = messages src conf init.d man

install-data-local: ltfs.pc
	if [ ! -d $(DESTDIR)$(libdir)/pkgconfig ]; then \
		mkdir -p $(DESTDIR)$(libdir)/pkgconfig; \
	fi
	cp ./ltfs.pc $(DESTDIR)$(libdir)/pkgconfig/

clean-local:
	rm -f ltfs.pc



# i know this isn't the right way to do this... but it's how i've been
# occasionally making .pkg files to install, now documented/automated
# to make life easier.
DEBREV=2
DEBVERSION=$(PACKAGE_VERSION)-$(DEBREV)
DEBNAME=$(PACKAGE_TARNAME)_$(DEBVERSION)
DEBFILE=$(abs_top_builddir)/$(DEBNAME).deb
DPKGDIR=$(abs_top_builddir)/$(DEBNAME)
DEBDIR=$(DPKGDIR)/DEBIAN
DEBCTRL=$(DEBDIR)/control
DEBSIZE=$(DEBDIR)/.size
DEBARCH=$(DEBDIR)/.arch
DEBDEPS=$(DEBDIR)/.deps
DEBCTRL_DEPS=$(DEBSIZE) $(DEBARCH) $(DEBDEPS)

.PHONY: dpkg
dpkg: $(DEBFILE)

$(DEBFILE): $(DEBCTRL)
	rm -f $(DEBCTRL_DEPS)
	dpkg-deb --build --root-owner-group $(DPKGDIR)
	rm -rf $(DPKGDIR)

$(DPKGDIR):
	./configure --prefix=/usr --sysconfdir=/etc --localstatedir=/var
	$(MAKE) clean
	$(MAKE) all
	$(MAKE) DESTDIR=$@ install-strip

$(DEBDIR): $(DPKGDIR)
	$(MKDIR_P) $@

$(DEBSIZE): $(DEBDIR)
	du -s $(DPKGDIR) | awk '{print $$1}' > $@

$(DEBARCH): $(DEBDIR)
	dpkg-architecture --query DEB_HOST_ARCH > $@

$(DEBDEPS): $(DEBDIR)
	$(srcdir)/debian/get_deps $(DPKGDIR) > $(DEBDIR)/.tmp
	awk -F": " '/^pkgs:/ {print $$2}' $(DEBDIR)/.tmp > $@
	rm -f $(DEBDIR)/.tmp

$(DEBCTRL): $(DEBCTRL_DEPS)
	cat $(srcdir)/debian/control | sed \
	    -e 's|__version__|$(DEBVERSION)|' \
	    -e "s|__size__|$$(cat $(DEBDIR)/.size)|" \
	    -e "s|__arch__|$$(cat $(DEBDIR)/.arch)|" \
	    -e "s|__deplibs__|$$(cat $(DEBDIR)/.deps)|" \
	    > $@
