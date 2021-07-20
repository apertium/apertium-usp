###############################################################################
## Makefile for apertium-usp
###############################################################################

LANG1=usp
BASENAME=apertium-$(LANG1)

TARGETS_COMMON =			\
	$(LANG1).automorf.hfst		\
	$(LANG1).automorf.bin		\
	$(LANG1).automorf.att.gz	\
	$(LANG1).autogen.hfst		\
	$(LANG1).autophon.hfst		\
	$(LANG1).autogen.bin		\
	$(LANG1).autogen.att.gz		\
	$(LANG1).autoseg.hfst		\
	$(LANG1).autoseg.bin		\
	$(LANG1).prob		\
	$(LANG1).rlx.bin

TARGETS_INTERMEDIATE = $(LANG1).mor.hfst $(LANG1).seg.hfst $(LANG1).gen.hfst

# This include defines goals for install-modes, .deps/.d and .mode files:
@ap_include@

###############################################################################

usp.automorf.hfst: usp.mor.hfst usp.spellrelax.hfst
	hfst-invert $< |  hfst-compose -F -1 - -2 usp.spellrelax.hfst  | hfst-invert | hfst-fst2fst -w -o $@

usp.autogen.hfst: usp.gen.hfst
	hfst-fst2fst -w $< -o $@

usp.autoseg.hfst: usp.seg.hfst 
	hfst-fst2fst -w $< -o $@

usp.mor.hfst: usp.gen.hfst usp.mor.twol.hfst
	hfst-compose-intersect -1 usp.gen.hfst -2 usp.mor.twol.hfst | hfst-realign | hfst-invert -o $@

# <impf><o_pl1><s_pl3>tʼis<v><tv>+o<mark>:k>oj>ki>tʼis>o
usp.gen.hfst: usp.lexc.hfst usp.twol.hfst
	hfst-compose-intersect -1 usp.lexc.hfst -2 usp.twol.hfst | hfst-realign -o $@

usp.twol.hfst: apertium-usp.usp.twol
	hfst-twolc $< -o $@

# <impf><o_pl1><s_pl3>tʼis<v><tv>+o<mark>:k>{o}j>k{i}>tʼis>o
usp.lexc.hfst: apertium-usp.usp.lexc
	hfst-lexc -A --Werror $< -o $@

usp.mor.twol.hfst: apertium-usp.usp.mor.twol
	hfst-twolc apertium-usp.usp.mor.twol -o usp.mor.twol.hfst

# kojkitʼiso:k>oj>ki>tʼis>o
usp.seg.hfst: usp.mor.hfst usp.gen.hfst
	hfst-compose -F -1 usp.mor.hfst -2 usp.gen.hfst -o usp.seg.hfst 

usp.phon.hfst: apertium-usp.usp.phon
	hfst-lexc apertium-usp.usp.phon -o $@

usp.autophon.hfst: usp.mor.hfst usp.phon.hfst
	hfst-fst2fst -w usp.phon.hfst -o usp.autophon.hfst
#	hfst-project -p upper usp.mor.hfst | hfst-compose -1 - -2 usp.phon.hfst | hfst-fst2fst -w -o usp.autophon.hfst

usp.spellrelax.hfst: apertium-usp.usp.spellrelax 
	hfst-regexp2fst -S -o $@ < $<

usp.rlx.bin: apertium-usp.usp.rlx
	cg-comp $< $@

$(LANG1).autogen.att.gz: $(LANG1).gen.hfst
	hfst-fst2txt $< | gzip -9 -c > $@

$(LANG1).automorf.att.gz: $(LANG1).mor.hfst usp.spellrelax.hfst
	hfst-invert $< |  hfst-compose -F -1 - -2 usp.spellrelax.hfst  | hfst-invert | hfst-fst2txt | gzip -9 -c > $@

$(LANG1).autoseg.att.gz: $(LANG1).seg.hfst
	hfst-fst2txt $< | gzip -9 -c > $@

$(LANG1).autogen.bin: $(LANG1).autogen.att.gz .deps/.d
	zcat < $< > .deps/$(LANG1).autogen.att
	lt-comp lr .deps/$(LANG1).autogen.att $@

$(LANG1).automorf.bin: $(LANG1).automorf.att.gz .deps/.d
	zcat < $< > .deps/$(LANG1).automorf.att
	lt-comp lr .deps/$(LANG1).automorf.att $@

$(LANG1).autoseg.bin: $(LANG1).autoseg.att.gz .deps/.d
	zcat < $< > .deps/$(LANG1).autoseg.att
	lt-comp lr .deps/$(LANG1).autoseg.att $@

$(LANG1).prob: $(BASENAME).$(LANG1).tagged
	apertium-tagger -s 0 -u 2 $@ $<

###############################################################################
## Distribution
###############################################################################

EXTRA_DIST=$(BASENAME).$(LANG1).lexc		\
		$(BASENAME).$(LANG1).twol	\
		$(BASENAME).$(LANG1).spellrelax \
		$(BASENAME).$(LANG1).rlx	\
		modes.xml

###############################################################################
## Installation stuff
###############################################################################
#
#   apertium_usp_dir: This is where the compiled binaries go
#   apertium_usp_srcdir: This is where the source files go

apertium_uspdir=$(prefix)/share/apertium/$(BASENAME)/
apertium_usp_srcdir=$(prefix)/share/apertium/$(BASENAME)/

EXTRA_TARGETS=

if HAVE_HFSTOSPELL
EXTRA_TARGETS += $(LANG1).zhfst
endif # HAVE_HFSTOSPELL

apertium_usp_DATA=$(TARGETS_COMMON) $(EXTRA_TARGETS) 

pkgconfigdir = $(prefix)/share/pkgconfig
pkgconfig_DATA = $(BASENAME).pc

noinst_DATA=modes/$(LANG1)-morph.mode

install-data-local: install-modes
	$(INSTALL_DATA) $(BASENAME).$(LANG1).lexc $(DESTDIR)$(apertium_usp_srcdir)
	$(INSTALL_DATA) $(BASENAME).$(LANG1).twol $(DESTDIR)$(apertium_usp_srcdir)
	$(INSTALL_DATA) $(BASENAME).$(LANG1).rlx $(DESTDIR)$(apertium_usp_srcdir)


###############################################################################
## Cleanup
###############################################################################

CLEANFILES = $(TARGETS_COMMON) $(TARGETS_INTERMEDIATE)
clean-local:
	-rm -rf .deps modes

###############################################################################

test: all
	apertium-regtest test
