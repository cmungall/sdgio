
GEMETS = backbone definitions groups skoscore
GEMETFS = $(patsubst %,gemet-%.rdf,$(GEMETS))

all: gemet summary-xrefs-gemet.txt xsummary.tsv no-align.tsv

gemet-backbone.rdf:
	wget http://www.eionet.europa.eu/gemet/gemet-backbone.rdf -O $@

gemet-definitions.rdf:
	wget 'http://www.eionet.europa.eu/gemet/gemet-definitions.rdf?langcode=en' -O $@

gemet-skoscore.rdf:
	wget 'http://www.eionet.europa.eu/gemet/gemet-skoscore.rdf?langcode=en' -O $@

gemet-groups.rdf:
	wget 'http://www.eionet.europa.eu/gemet/gemet-groups.rdf?langcode=en' -O $@

gemet-all.rdf: $(GEMETFS)
	rdfcat $^ > $@

gemet-full.obo: gemet-all.rdf
	./translate-gemet-rdf.py > $@.tmp && hacky-fix-gemet-ids.pl $@.tmp > $@.tmp2 && owltools $@.tmp2 -o -f obo $@

# This is the version from the spreadsheet
gemet.obo: priority-gemet.tsv 
	perl -npe 's//GEMET:/' $< | cut -f1,2 | tbl2obo.pl  > $@

gemet-labels.tsv: priority-gemet.tsv
	cut -f2 $<  > $@

#GEMETX = chebi
GEMETX = mondo oba mesh sctid chebi eo envo to pato exo
XOBOS = $(patsubst %, xrefs-gemet-%.obo,$(GEMETX))
LABELS = $(patsubst %, labels-%.tsv,$(GEMETX))

gemet: $(XOBOS) 

align-gemet-%.tsv: gemet.obo
	blip-findall -debug index -i ignore_gemet.pro -i $< -r $* -u metadata_nlp -goal index_entity_pair_label_match "class(X),id_idspace(X,'GEMET'),entity_pair_label_reciprocal_best_intermatch(X,Y,S)" -select "x(X,Y,S)" -label -use_tabs -no_pred > $@.tmp && mv $@.tmp $@
.PRECIOUS: align-gemet-%.tsv

xrefs-%.obo: align-%.tsv
	cut -f1-4 $< | sort -u | tbl2obolinks.pl --rel xref - > $@

allx.obo: $(XOBOS)
	obo-cat.pl $^ > $@
#	owltools $^ --merge-support-ontologies -o -f obo $@

xsummary.tsv: allx.obo
	blip-findall -i $< -i gemet.obo -u xref_util prefixset_stats/2 -no_pred > $@.tmp &&  mysort -s -k2 -n -r $@.tmp > $@

gemet-bestmatches.tsv: allx.obo
	blip-findall -i $< -i gemet.obo -consult sdgutil gemet_rpt/3 -no_pred > $@.tmp && sort -u $@.tmp > $@

summary-xrefs-%.txt:
	grep -c ^id: xrefs-$*-*.obo

no-align.tsv: allx.obo gemet.obo
	blip-findall -i $< -i gemet.obo "class(X),id_idspace(X,'GEMET'),\+entity_xref(X,_)" -select X -label -use_tabs | sort -u > $@

labels: $(LABELS)
labels-%.tsv:
	blip-findall -r $* -consult synreport.pro rpt/3 -grid -use_tabs -no_pred > $@
