# Makefile in documentation directory
#
# This file is part of DRBD by Philipp Reisner and Lars Ellenberg.
#
# drbd is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2, or (at your option)
# any later version.
#
# drbd is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with drbd; see the file COPYING.  If not, write to
# the Free Software Foundation, 675 Mass Ave, Cambridge, MA 02139, USA.

# variables set by configure
DISTRO = 
prefix = /usr
exec_prefix = /usr
localstatedir = /var
datarootdir = /usr/share
datadir = /usr/share
sbindir = /usr/sbin
sysconfdir = /etc
mandir = /usr/share/man
BASH_COMPLETION_SUFFIX = 
UDEV_RULE_SUFFIX = 
INITDIR = /etc/init.d
LIBDIR = /usr/lib/drbd
CC = gcc
CFLAGS = -g -O2
XSLTPROC = /usr/bin/xsltproc

# features enabled or disabled by configure
WITH_83_SUPPORT = yes
WITH_84_SUPPORT = yes
WITH_UDEV = yes
WITH_XEN = yes
WITH_PACEMAKER = yes
WITH_HEARTBEAT = yes
WITH_RGMANAGER = no
WITH_BASHCOMPLETION = yes

# variables meant to be overridden from the make command line
DESTDIR ?= /

# Needed for pattern substitution
SHELL=/bin/bash

MANPAGES       := drbdsetup.8 drbd.conf.5 drbd.8 drbdadm.8 drbdmeta.8
ifeq ($(WITH_HEARTBEAT),yes)
MANPAGES += drbddisk.8
endif

STYLESHEET_PREFIX ?= http://docbook.sourceforge.net/release/xsl/current
MANPAGES_STYLESHEET ?= $(STYLESHEET_PREFIX)/manpages/docbook.xsl
HTML_STYLESHEET ?= $(STYLESHEET_PREFIX)/xhtml/docbook.xsl
FO_STYLESHEET ?= $(STYLESHEET_PREFIX)/fo/docbook.xsl

XSLTPROC_OPTIONS ?= --xinclude
XSLTPROC_OPTIONS += --stringparam variablelist.term.break.after 1
#XSLTPROC_OPTIONS += --stringparam variablelist.term.separator ""
XSLTPROC_MANPAGES_OPTIONS ?= $(XSLTPROC_OPTIONS)
XSLTPROC_HTML_OPTIONS ?= $(XSLTPROC_OPTIONS)
XSLTPROC_FO_OPTIONS ?= $(XSLTPROC_OPTIONS)

DRBDSETUP_CMDS = new-resource new-minor del-resource del-minor
DRBDSETUP_CMDS += attach connect disk-options net-options resource-options
DRBDSETUP_CMDS += disconnect detach primary secondary verify invalidate invalidate-remote
DRBDSETUP_CMDS += down wait-connect wait-sync role cstate dstate
DRBDSETUP_CMDS += resize check-resize pause-sync resume-sync
DRBDSETUP_CMDS += outdate show-gi get-gi show events events2
DRBDSETUP_CMDS += status suspend-io resume-io new-current-uuid

make_doc := $(shell $(XSLTPROC)				\
	$(XSLTPROC_MANPAGES_OPTIONS)			\
	$(MANPAGES_STYLESHEET) < /dev/null > /dev/null 2>&1 && echo doc )

ifeq ($(make_doc),doc)
all: doc
else
all:
	@echo "To (re)make the documentation: make doc"
endif

clean:
	@echo "To clean the documentation: make doc-clean"

.PHONY: all clean doc man doc-clean distclean
.PHONY: install uninstall html pdf ps
ifeq ($(WITH_84_SUPPORT),yes)
doc:	man
else
doc:
endif


doc-clean: distclean

####### Implicit rules

.SUFFIXES: .sgml .5 .8 .html .pdf .ps

%.5 %.8: %.xml
	$(XSLTPROC) \
	$(XSLTPROC_MANPAGES_OPTIONS) \
	$(MANPAGES_STYLESHEET) $<

%.html: %.xml
	$(XSLTPROC) -o $@ \
	$(XSLTPROC_HTML_OPTIONS) \
	$(HTML_STYLESHEET) $<

%.fo: %.xml
	$(XSLTPROC) -o $@ \
	$(XSLTPROC_FO_OPTIONS) \
	$(FO_STYLESHEET) $<

../../user/v84/drbdsetup.o: FORCE
	$(MAKE) -C $(@D) drbdsetup-84

.PHONY: FORCE
FORCE:

# Don't try to re-make files tracked in git
FILES_IN_GIT := # $(shell git ls-files)
FILES_IN_GIT += Makefile.in drbd.conf.xml drbd.xml drbdadm.xml drbddisk.xml
FILES_IN_GIT += drbdmeta.xml drbdsetup.xml xml-usage-to-docbook.xsl
$(FILES_IN_GIT): ;


drbdsetup_X.xml := $(patsubst %,drbdsetup_%.xml,$(DRBDSETUP_CMDS))
drbdsetup_xml-help_X.xml := $(patsubst %,drbdsetup_xml-help_%.xml,$(DRBDSETUP_CMDS))

$(drbdsetup_xml-help_X.xml): ../../user/v84/drbdsetup.o
$(drbdsetup_X.xml): xml-usage-to-docbook.xsl

drbdsetup_xml-help_%.xml:
	../../user/v84/drbdsetup-84 xml-help $* > $@

drbdsetup_%.xml: drbdsetup_xml-help_%.xml
	$(XSLTPROC) -o $@ xml-usage-to-docbook.xsl $<

distclean:
ifeq ($(make_doc),doc)
	rm -f *.[58] manpage.links manpage.refs *~ manpage.log
endif
	rm -f *.ps.gz *.pdf *.ps *.html pod2htm*
	rm -f drbdsetup_*.xml

#######

man:	$(MANPAGES)

install:
ifeq ($(WITH_84_SUPPORT),yes)
	@ok=true; for f in $(MANPAGES) ; \
		do [ -e $$f ] || { echo $$f missing ; ok=false; } ; \
	done ; $$ok
	set -e; for f in $(MANPAGES) ; do \
		s=$${f##*.}; \
		b=$${f%.[0-9]}; \
		install -v -D -m 644 $$f $(DESTDIR)$(mandir)/man$$s/$$b-8.4.$$s ; \
	done
endif

uninstall:
ifeq ($(WITH_84_SUPPORT),yes)
	@ set -e; for f in $(MANPAGES) ; do \
		s=$${f##*.}; \
		b=$${f%.[0-9]}; \
		rm -vf $(DESTDIR)$(mandir)/man$$s/$$b-8.4.$$s ; \
	done
endif

html:	$(MANPAGES:.8=.html)

pdf:	$(MANPAGES:.8=.pdf)

ps:	$(MANPAGES:.8=.ps)

drbdsetup.8: drbdsetup.xml $(drbdsetup_X.xml)

.PHONY: install uninstall clean distclean
../../configure:
	@echo "please (re-)run ./autogen.sh with appropriate arguments"; exit 1
../../config.status: ../../configure
	@echo "please (re-)run ./configure with appropriate arguments"; exit 1
Makefile: Makefile.in ../../config.status
	cd ../.. && ./config.status documentation/v84/Makefile
