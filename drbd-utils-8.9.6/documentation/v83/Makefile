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
SOURCES        := $(wildcard *.xml)

STYLESHEET_PREFIX ?= http://docbook.sourceforge.net/release/xsl/current
MANPAGES_STYLESHEET ?= $(STYLESHEET_PREFIX)/manpages/docbook.xsl
HTML_STYLESHEET ?= $(STYLESHEET_PREFIX)/xhtml/docbook.xsl
FO_STYLESHEET ?= $(STYLESHEET_PREFIX)/fo/docbook.xsl

XSLTPROC_OPTIONS ?= --xinclude
XSLTPROC_MANPAGES_OPTIONS ?= $(XSLTPROC_OPTIONS)
XSLTPROC_HTML_OPTIONS ?= $(XSLTPROC_OPTIONS)
XSLTPROC_FO_OPTIONS ?= $(XSLTPROC_OPTIONS)

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

ifeq ($(WITH_83_SUPPORT),yes)
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

distclean:
	rm -f *.[58] manpage.links manpage.refs *~ manpage.log
	rm -f *.ps.gz *.pdf *.ps *.html pod2htm*

#######

man:	$(MANPAGES)

install:
ifeq ($(WITH_83_SUPPORT),yes)
	@ok=true; for f in $(MANPAGES) ; \
		do [ -e $$f ] || { echo $$f missing ; ok=false; } ; \
	done ; $$ok
	set -e; for f in $(MANPAGES) ; do \
		s=$${f##*.}; \
		b=$${f%.[0-9]}; \
		install -v -D -m 644 $$f $(DESTDIR)$(mandir)/man$$s/$$b-8.3.$$s ; \
	done
endif

uninstall:
ifeq ($(WITH_83_SUPPORT),yes)
	@ set -e; for f in $(MANPAGES) ; do \
		s=$${f##*.}; \
		b=$${f%.[0-9]}; \
		rm -v $(DESTDIR)$(mandir)/man$$s/$$b-8.3.$$s ; \
	done
endif

html:	$(SOURCES:.xml=.html)

pdf:	$(SOURCES:.xml=.pdf)

ps:	$(SOURCES:.xml=.ps)

../../configure:
	@echo "please (re-)run ./autogen.sh with appropriate arguments"; exit 1
../../config.status: ../../configure
	@echo "please (re-)run ./configure with appropriate arguments"; exit 1
Makefile.in: ;
Makefile: Makefile.in ../../config.status
	cd ../.. && ./config.status documentation/v83/Makefile
