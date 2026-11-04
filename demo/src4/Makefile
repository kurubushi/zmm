install: build/big-ugly
	install -d $(DESTDIR)/.rhodes
	sed 's/@USER@/$(USER)/' config/castle-3.in >$(DESTDIR)/.rhodes/castle-3
	install -m 644 config/lancet-2.$(shell uname -s) $(DESTDIR)/.rhodes/lancet-2
	install -m 644 build/big-ugly $(DESTDIR)/.rhodes/big-ugly

build/big-ugly:
	mkdir -p build
	curl -fsSL -o build/big-ugly https://raw.githubusercontent.com/kurubushi/zmm/master/demo/acahualla/big-ugly

clean:
	rm -rf build

# ~/.zmm-demo/src
# ├── Makefile
# └── config
#     ├── castle-3.in
#     ├── lancet-2.Darwin
#     └── lancet-2.Linux
