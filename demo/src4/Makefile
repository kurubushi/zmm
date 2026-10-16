# zmm replaces $DESTDIR with $HOME.

# Linux, Darwin, ...
OS := $(shell uname -s)

# User name
# $ echo $USER # doctor

# From the environment, if any ("AUTOPILOT=on zmm install")
AUTOPILOT ?= off

install: build/lancet-2
	install -d $(DESTDIR)/.rhodes
	sed 's/@USER@/$(USER)/' config/castle-3.in >$(DESTDIR)/.rhodes/castle-3
	install -m 644 build/lancet-2 $(DESTDIR)/.rhodes/lancet-2
	sed 's/@AUTOPILOT@/$(AUTOPILOT)/' config/big-ugly.in >$(DESTDIR)/.rhodes/big-ugly

# Join the common part and the part for the OS
# (make joins them again only when a part changes)
build/lancet-2: config/lancet-2/common config/lancet-2/$(OS)
	mkdir -p build
	cat config/lancet-2/common config/lancet-2/$(OS) >build/lancet-2

# For "zmm install --clean"
clean:
	rm -rf build

# ~/.zmm-demo/src
# ├── Makefile
# └── config
#     ├── big-ugly.in
#     ├── castle-3.in
#     └── lancet-2
#         ├── Darwin
#         ├── Linux
#         └── common
