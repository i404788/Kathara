#!/usr/bin/make

VERSION=2.0.3
LAUNCHPAD_NAME=user

ARCHITECTURE:=$(shell dpkg --print-architecture)

.PHONY: debUnsigned debSigned deps clean dockerSignedDebian dockerUnsignedDebian docker-build-debian dockerSignedUbuntu dockerUnsignedUbuntu docker-build-ubuntu compile ppa prepareDeb

docker-build-debian:
	cd Docker-Linux-Build-Debian-9 && docker build -t kathara/debian-9-build .

dockerSignedDebian: docker-build-debian
	docker run -ti --rm -v $(HOME)/.gnupg:/root/.gnupg -v `pwd`/../../:/opt/kathara kathara/debian-9-build /bin/bash -c "make clean && make deps && make prepareDebForDebian && make debSigned && make ppa"

dockerUnsignedDebian: docker-build-debian
	docker run -ti --rm -v `pwd`/../../:/opt/kathara kathara/debian-9-build /bin/bash -c "make clean && make deps && make prepareDebForDebian && make debUnsigned"

docker-build-ubuntu:
	cd Docker-Linux-Build-Ubuntu-1904 && docker build -t kathara/ubuntu-1904-build .

dockerSignedUbuntu: docker-build-ubuntu
	docker run -ti --rm -v $(HOME)/.gnupg:/root/.gnupg -v `pwd`/../../:/opt/kathara kathara/ubuntu-1904-build /bin/bash -c "make clean && make deps && make prepareDebForUbuntu && make debSigned && make ppa"

dockerUnsignedUbuntu: docker-build-ubuntu
	docker run -ti --rm -v `pwd`/../../:/opt/kathara kathara/ubuntu-1904-build /bin/bash -c "make clean && make deps && make prepareDebForUbuntu && make debUnsigned"

compile:
	cd ../../src && python3 -m nuitka --plugin-enable=pylint-warnings --follow-imports --standalone --include-plugin-directory=Resources kathara.py
	rm -f ../../src/kathara.dist/libbz2.so.1.0
	rm -f ../../src/kathara.dist/libexpat.so.1
	rm -f ../../src/kathara.dist/libtinfo.so.6
	rm -f ../../src/kathara.dist/libz.so.1
	rm -f ../../src/kathara.dist/libtinfo.so.5
	rm -f ../../src/kathara.dist/libcrypto.so.1.1

prepareDeb: compile
	mkdir -p Output/kathara-$(VERSION)
	cp -r ../../src/kathara.dist Output/kathara-$(VERSION)
	cd Output && tar -zcf kathara_$(VERSION).orig.tar.gz kathara-$(VERSION)
	cp -r debian Output/kathara-$(VERSION)/
	cd ../../docs && make roff-build
	cp -r ../../docs/Roff Output/kathara-$(VERSION)/debian/manpages
	cd ../../docs && make cleanall
	ls Output/kathara-$(VERSION)/debian/manpages/ | awk '{print "debian/manpages/"$$0}' > Output/kathara-$(VERSION)/debian/kathara.manpages
	sed -i -e "s|__DATE__|$$(date -R)|g" Output/kathara-$(VERSION)/debian/changelog

prepareDebForDebian: prepareDeb
	sed -i -e 's/__UBUNTU_VERSION__/bionic/g' Output/kathara-$(VERSION)/debian/changelog

prepareDebForUbuntu: prepareDeb
	sed -i -e 's/__UBUNTU_VERSION__/disco/g' Output/kathara-$(VERSION)/debian/changelog

debSigned:
	cd Output/kathara-$(VERSION); debuild -S -sa
	rm -Rf Output/kathara-$(VERSION)

debUnsigned:
	cd Output/kathara-$(VERSION); debuild -uc -us
	rm -Rf Output/kathara-$(VERSION)

ppa:
	sed -i -e 's/__LAUNCHPAD_NAME__/$(LAUNCHPAD_NAME)/g' $(HOME)/.dput.cf
	dput -u ppa:katharaframework/kathara Output/*.changes

deps:
	python3 -m pip install nuitka
	cd ../../src && python3 -m pip install -r requirements.txt
	gem install ronn-ng

clean:
	rm -rf ../../src/kathara.build
	rm -rf ../../src/kathara.dist
	find ../../src | grep -E "(__pycache__|\.pyc|\.pyo)$$" | xargs rm -rf
	if [ -d Output ]; then \
		rm -r Output; \
	fi