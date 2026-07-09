TEX := freshness-ietf126.tex
PDF := $(TEX:.tex=.pdf)

.PHONY: all clean

all: $(PDF)

$(PDF): $(TEX)
	latexmk -pdf -interaction=nonstopmode -halt-on-error $(TEX)

clean:
	latexmk -C $(TEX)
