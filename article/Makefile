LATEXMK := latexmk -r latexmkrc

.PHONY: all watch clean distclean

all:
	$(LATEXMK) main.tex

watch:
	$(LATEXMK) -pvc -view=none main.tex

clean:
	$(LATEXMK) -c main.tex

distclean:
	$(LATEXMK) -C main.tex
	rm -rf build
