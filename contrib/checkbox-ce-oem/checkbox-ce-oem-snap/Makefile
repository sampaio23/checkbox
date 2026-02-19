SNAP_VERSION = 1.0

ifndef SERIES
$(error SERIES is not set)
endif

ifndef RELEASE
$(error RELEASE is not set)
endif

SNAP_OUTPUT = snapcraft.yaml
SNAP_WORK_FOLDER = series_$(SERIES)$(RELEASE)
SNAP_TEMPLATE = snapcraft.yaml.in

SNAPBASE := $(shell \
    if [ "$(RELEASE)" = "16" ]; then \
        echo "core"; \
    elif [ "$(RELEASE)" = "18" ]; then \
        echo "core18"; \
    elif [ "$(RELEASE)" = "20" ]; then \
        echo "core20"; \
    elif [ "$(RELEASE)" = "22" ]; then \
        echo "core22"; \
    elif [ "$(RELEASE)" = "24" ]; then \
        echo "core24"; \
    else \
        echo ""; \
    fi \
)

ifndef SNAPBASE
$(error RELEASE is not valid: pick between 16, 18, 20, 22, or 24)
endif

PYTHONVER := $(shell \
    if [ "$(SNAPBASE)" = "core" ]; then \
        echo "python3.5"; \
    elif [ "$(SNAPBASE)" = "core18" ]; then \
        echo "python3.6"; \
    elif [ "$(SNAPBASE)" = "core20" ]; then \
        echo "python3.8"; \
    elif [ "$(SNAPBASE)" = "core22" ]; then \
        echo "python3.10"; \
    elif [ "$(SNAPBASE)" = "core24" ]; then \
        echo "python3.12"; \
    else \
        echo ""; \
    fi \
)

ifndef PYTHONVER
$(error SNAPBASE is not valid: pick between core, core18, core20, core22, or core24)
endif

SERIESNAME := $(shell \
    if [ "$(SNAPBASE)" = "core" ]; then \
        echo "xenial"; \
    elif [ "$(SNAPBASE)" = "core18" ]; then \
        echo "bionic"; \
    elif [ "$(SNAPBASE)" = "core20" ]; then \
        echo "focal"; \
    elif [ "$(SNAPBASE)" = "core22" ]; then \
        echo "jammy"; \
    elif [ "$(SNAPBASE)" = "core24" ]; then \
        echo "noble"; \
    else \
        echo ""; \
    fi \
)

SNAPCRAFT_FOLDER_PREFIX := $(shell \
    if [ "$(SNAPBASE)" = "core" ]; then \
        echo "SNAP"; \
    elif [ "$(SNAPBASE)" = "core18" ]; then \
        echo "SNAP"; \
    elif [ "$(SNAPBASE)" = "core20" ]; then \
        echo "SNAP"; \
    elif [ "$(SNAPBASE)" = "core22" ]; then \
        echo "SNAP"; \
    elif [ "$(SNAPBASE)" = "core24" ]; then \
        echo ""; \
    else \
        echo ""; \
    fi \
)

OVERRIDE_BUILD_COMMAND := $(shell \
    if [ "$(SNAPBASE)" = "core" ]; then \
        echo "SNAP"; \
    elif [ "$(SNAPBASE)" = "core18" ]; then \
        echo "SNAP"; \
    elif [ "$(SNAPBASE)" = "core20" ]; then \
        echo "snapcraftctl build"; \
    elif [ "$(SNAPBASE)" = "core22" ]; then \
        echo "snapcraftctl build"; \
    elif [ "$(SNAPBASE)" = "core24" ]; then \
        echo "craftctl default"; \
    else \
        echo ""; \
    fi \
)

OVERRIDE_PRIME_COMMAND := $(shell \
    if [ "$(SNAPBASE)" = "core20" ]; then \
        echo "snapcraftctl prime"; \
    elif [ "$(SNAPBASE)" = "core22" ]; then \
        echo "snapcraftctl prime"; \
    elif [ "$(SNAPBASE)" = "core24" ]; then \
        echo "craftctl default"; \
    else \
        echo ""; \
    fi \
)

ifeq ($(SERIES),classic)
    ifeq ($(RELEASE),$(filter $(RELEASE),22 20))
	define OVERRIDE_PRIME_COMMAND
      snapcraftctl prime
        endef
    else ifeq ($(RELEASE),24)
	define OVERRIDE_PRIME_COMMAND
      craftctl default
        endef
    endif
else ifeq ($(SERIES),uc)
    ifeq ($(RELEASE),$(filter $(RELEASE),22 20))
	define OVERRIDE_PRIME_COMMAND
      snapcraftctl prime
      rm lib/systemd/system/alsa-utils.service
        endef
    else ifeq ($(RELEASE),24)
	define OVERRIDE_PRIME_COMMAND
      craftctl default
      rm lib/systemd/system/alsa-utils.service
        endef
    endif
endif


CHECKBOX_RUNTIME := $(shell \
    if [ "$(SNAPBASE)" = "core" ]; then \
        echo "checkbox16"; \
    elif [ "$(SNAPBASE)" = "core18" ]; then \
        echo "checkbox18"; \
    elif [ "$(SNAPBASE)" = "core20" ]; then \
        echo "checkbox20"; \
    elif [ "$(SNAPBASE)" = "core22" ]; then \
        echo "checkbox22"; \
    elif [ "$(SNAPBASE)" = "core24" ]; then \
        echo "checkbox24"; \
    else \
        echo ""; \
    fi \
)

APPEND_SUMMARY := $(shell \
    if [ "$(SERIES)" = "classic" ]; then \
        echo " for $(RELEASE).04 classic"; \
    elif [ "$(SERIES)" = "uc" ]; then \
        echo ""; \
    else \
        echo ""; \
    fi \
)

ifeq ($(SERIES),classic)
    ifeq ($(RELEASE),$(filter $(RELEASE),24 22))
	define PROVIDER_EXTRA_PACKAGES
      - python3-rpyc
        endef
    endif
else ifeq ($(SERIES),uc)
    define PROVIDER_EXTRA_PACKAGES
      - python3-rpyc
      - alsa-utils
    endef
endif

ifeq ($(SERIES),uc)
    define EXTRA_CONFIG
layout:
  # This is an workaround to access alsa config on core image
  /usr/share/alsa:
    bind: $$SNAP/usr/share/alsa
    endef
endif

export SERIESNAME
export SNAPBASE
export SNAPCRAFT_FOLDER_PREFIX
export SNAP_VERSION
export OVERRIDE_BUILD_COMMAND
export OVERRIDE_PRIME_COMMAND
export PYTHONVER
export CHECKBOX_RUNTIME
export APPEND_SUMMARY

export PROVIDER_EXTRA_PACKAGES
export EXTRA_CONFIG

$(SNAP_OUTPUT): $(SNAP_TEMPLATE)
	mv $(SNAP_WORK_FOLDER)/snap/snapcraft.yaml $(SNAP_WORK_FOLDER)/snap/snapcraft.yaml.bkp
	./prepare.sh $(SNAP_WORK_FOLDER)
	envsubst '$$CHECKBOX_RUNTIME $$SERIESNAME $$SNAPBASE $$SNAPCRAFT_FOLDER_PREFIX $$SNAP_VERSION $$OVERRIDE_BUILD_COMMAND $$OVERRIDE_PRIME_COMMAND $$PYTHONVER $$APPEND_SUMMARY $$PROVIDER_EXTRA_PACKAGES $$EXTRA_CONFIG' < $< > $(SNAP_WORK_FOLDER)/snap/$@

all: $(SNAP_OUTPUT)

clean:
	rm -rf $(SNAP_WORK_FOLDER)

.PHONY: clean $(SNAP_OUTPUT)
