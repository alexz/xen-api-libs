CC = gcc
CFLAGS = -Wall -fPIC -O2 -I/usr/lib/ocaml
OCAMLC = ocamlc -g
OCAMLOPT = ocamlopt

FEPP = camlp4o -I ../rpc-light -I $(shell ocamlfind query type-conv) pa_type_conv.cmo pa_rpc.cma

LDFLAGS = -cclib -L./

VERSION := $(shell git show-ref --head | grep -E " HEAD$$" | cut -f 1 -d " " || echo 0)
OCAMLOPTFLAGS = -g -dtypes

OCAMLABI := $(shell ocamlc -version)
OCAMLLIBDIR := $(shell ocamlc -where)
OCAMLDESTDIR ?= $(OCAMLLIBDIR)

OBJS = tapctl
INTF = $(foreach obj, $(OBJS),$(obj).cmi)
LIBS = tapctl.cma tapctl.cmxa

DOCDIR = /myrepos/xen-api-libs/doc

OCAMLFLAGS = -pp '${FEPP}' -I ../rpc-light -I ../stdext


all: $(INTF) $(LIBS) $(PROGRAMS)

bins: $(PROGRAMS)

libs: $(LIBS)

tapctl.cmxa: $(foreach obj,$(OBJS),$(obj).cmx)
	$(OCAMLOPT) $(OCAMLOPTFLAGS) $(OCAMLFLAGS) -a -o $@ $(foreach obj,$(OBJS),$(obj).cmx)

tapctl.cma: $(foreach obj,$(OBJS),$(obj).cmo)
	$(OCAMLC) $(OCAMLOPTFLAGS) $(OCAMLFLAGS) -a -o $@ $(foreach obj,$(OBJS),$(obj).cmo)

%.cmo: %.ml
	$(OCAMLC) $(OCAMLFLAGS) -c -o $@ $<

%.cmi: %.mli
	$(OCAMLC) $(OCAMLFLAGS) -c -o $@ $<

%.cmx: %.ml
	$(OCAMLOPT) $(OCAMLOPTFLAGS) $(OCAMLFLAGS) -c -o $@ $<

%.o: %.c
	$(CC) $(CFLAGS) -c -o $@ $<

META: META.in
	sed 's/@VERSION@/$(VERSION)/g' < $< > $@

.PHONY: install
install: path = $(DESTDIR)$(shell ocamlfind printconf destdir)
install: $(LIBS) META
	mkdir -p $(path)
	ocamlfind install -destdir $(path) -ldconf ignore tapctl META $(INTF) $(LIBS) *.a *.cmx *.sp?t

.PHONY: uninstall
uninstall:
	ocamlfind remove tapctl

.PHONY: doc
doc: $(INTF)
	python ../doc/doc.py $(DOCDIR) "tapctl" "package" "$(OBJS)" "." "" ""

clean:
	rm -f *.o *.so *.a *.cmo *.cmi *.cma *.cmx *.cmxa *.annot *.sp?t $(LIBS) $(PROGRAMS)
