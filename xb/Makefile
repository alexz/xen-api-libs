CC = gcc
CFLAGS = -Wall -fPIC -O2 -I/usr/lib/ocaml -I$(XEN_ROOT)/usr/include -I../mmap
OCAMLC = ocamlc -g -I ../mmap
OCAMLOPT = ocamlopt
OCAMLOPTFLAGS = -g -dtypes -I ../mmap

LDFLAGS = -cclib -L./

VERSION := $(shell git show-ref --head | grep -E " HEAD$$" | cut -f 1 -d " " || echo 0)

OCAMLABI := $(shell ocamlc -version)
OCAMLLIBDIR := $(shell ocamlc -where)
OCAMLDESTDIR ?= $(OCAMLLIBDIR)

PREINTF = op.cmi partial.cmi packet.cmi
PREOBJS = op partial packet xs_ring
PRELIBS = $(foreach obj, $(PREOBJS),$(obj).cmo) $(foreach obj,$(PREOJBS),$(obj).cmx)
OBJS = op partial packet xs_ring xb
INTF = op.cmi packet.cmi xb.cmi
LIBS = xb.cma xb.cmxa

DOCDIR = /myrepos/xen-api-libs/doc

all: $(PREINTF) $(PRELIBS) $(INTF) $(LIBS) $(PROGRAMS)

bins: $(PROGRAMS)

libs: $(LIBS)

xb.cmxa: libxb_stubs.a $(foreach obj,$(OBJS),$(obj).cmx)
	$(OCAMLOPT) $(OCAMLOPTFLAGS) -a -o $@ -cclib -lxb_stubs $(foreach obj,$(OBJS),$(obj).cmx)

xb.cma: $(foreach obj,$(OBJS),$(obj).cmo)
	$(OCAMLC) -a -dllib dllxb_stubs.so -cclib -lxb_stubs -o $@ $(foreach obj,$(OBJS),$(obj).cmo)

xb_stubs.a: xs_ring_stubs.o xb_stubs.o
	ocamlmklib -o xb_stubs $+

libxb_stubs.a: xs_ring_stubs.o xb_stubs.o
	ar rcs $@ $+
	ocamlmklib -o xb_stubs $+

%.cmo: %.ml
	$(OCAMLC) -c -o $@ $<

%.cmi: %.mli
	$(OCAMLC) -c -o $@ $<

%.mli: %.ml
	$(OCAMLC) -i $< > $@

%.cmx: %.ml
	$(OCAMLOPT) $(OCAMLOPTFLAGS) -c -o $@ $<

%.o: %.c
	$(CC) $(CFLAGS) -c -o $@ $<

META: META.in
	sed 's/@VERSION@/$(VERSION)/g' < $< > $@

.PHONY: install
install: path = $(DESTDIR)$(shell ocamlfind printconf destdir)
install: $(LIBS) META
	mkdir -p $(path)
	ocamlfind install -destdir $(path) -ldconf ignore xb META $(INTF) $(LIBS) *.a *.so *.cmx *.sp?t

.PHONY: uninstall
uninstall:
	ocamlfind remove xb

.PHONY: doc
doc: $(INTF)
	python ../doc/doc.py $(DOCDIR) "xb" "package" "$(OBJS)" "." "mmap" ""

clean:
	rm -f *.o *.so *.a *.cmo *.cmi *.cma *.cmx *.cmxa *.annot *.sp?t $(LIBS) $(PROGRAMS)

