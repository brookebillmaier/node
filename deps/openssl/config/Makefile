# This Makefile is confirmed to be run only on Linux (CentOS and
# Ubuntu). perl5 and gas(>=2.26) are needed.

UNAME_S := $(shell uname -s)
ifneq ($(UNAME_S),Linux)
  $(error This can be run only on Linux)
endif

PERL    = perl

# Supported architecture list
ARCHS = aix-gcc aix64-gcc BSD-x86_64 \
darwin64-x86_64-cc darwin-i386-cc linux-aarch64 \
linux-armv4 linux-elf linux-x32 linux-x86_64 linux-ppc \
linux-ppc64 linux-ppc64le linux32-s390x linux64-s390x \
solaris-x86-gcc solaris64-x86_64-gcc VC-WIN64A VC-WIN32

CONFIGURE = ./Configure
# no-comp: against CRIME attack
# no-shared: openssl-cli needs static link
# no-afalgeng: old Linux kernel < 4.0 does not support it
COPTS =  no-comp no-shared no-afalgeng

# disable platform check in Configure
NO_WARN_ENV = CONFIGURE_CHECKER_WARN=1

GITIGNORE = $(OPSSL_SRC)/.gitignore
GENERATE = ./generate_gypi.pl

OPSSL_SRC = ../openssl

# Header files generated with Configure
CFG = opensslconf.h
SRC_CFG = $(OPSSL_SRC)/include/openssl/$(CFG)
INT_CFGS = bn_conf.h dso_conf.h
INT_CFG_DIR = $(OPSSL_SRC)/crypto/include/internal

PHONY = all clean replace
.PHONY: $(PHONY)

all: $(ARCHS) replace

# Configure and generate openssl asm files for each archs
$(ARCHS):
# Remove openssl .gitignore to follow nodejs .gitignore
	if [ -e $(GITIGNORE) ]; then rm $(GITIGNORE); fi
# Confgure asm and generate asm sources
	cd $(OPSSL_SRC); $(NO_WARN_ENV) $(PERL) $(CONFIGURE) $(COPTS) $@;
	$(PERL) -w -I$(OPSSL_SRC) $(GENERATE) asm $@
# Confgure no-asm and generate no-asm sources
	cd $(OPSSL_SRC); $(NO_WARN_ENV) $(PERL) $(CONFIGURE) $(COPTS) \
	no-asm $@;
	$(PERL) -w -I$(OPSSL_SRC) $(GENERATE) no-asm $@

# Replace and copy arch dependent headers
replace:
	cp ./$(CFG).tmpl $(SRC_CFG)
	@for c in $(INT_CFGS); do \
	    cp ./$$c.tmpl $(INT_CFG_DIR)/$$c; \
	done

clean:
	find archs \( -name \*.S -o -name \*.s -o -name \*.asm -o \
	-name \*.gypi -o -name \*.h -o -name \*.pm \) -exec rm "{}" \;
