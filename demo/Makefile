.PHONY: install uninstall

big-ugly:
	./gen_big_ugly.sh

install: big-ugly
	install -d $(HOME)/.zmm-demo 
	install -m 644 src/* $(HOME)/.zmm-demo
	install -m 644 big-ugly $(HOME)/.zmm-demo

uninstall:
	rm -rf $(HOME)/.zmm-demo
