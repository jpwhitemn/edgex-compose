# /*******************************************************************************
#  * Copyright 2021 Intel
#  *
#  * Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except
#  * in compliance with the License. You may obtain a copy of the License at
#  *
#  * http://www.apache.org/licenses/LICENSE-2.0
#  *
#  * Unless required by applicable law or agreed to in writing, software distributed under the License
#  * is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express
#  * or implied. See the License for the specific language governing permissions and limitations under
#  * the License.
#  *
#  *******************************************************************************/

.PHONY: help portainer portainer-down \
	build compose run pull gen \
	build-taf taf-compose taf-compose-perf \
	get-token upload-tls-cert get-consul-acl-token \
	ui-down down clean
.SILENT: help get-token del-token upload-tls-cert get-consul-acl-token

include .env

COMPOSE_FILES:=-f docker-compose-base.yml
TOKEN_LIST:=""
KNOWN_SECRETS_LIST:="redisdb[app-rules-engine]"
GEN_EXT_DIR:=gen_ext_scty

# Must have spaces around words for `filter-out` function to work properly.
# Dashes at beginning & end of line ensure space before/after the first/last option on that line
# and don't impact the option list
define OPTIONS
 - arm64 no-secty dev app-dev -
 - zmq-bus mqtt-bus mqtt-broker -
 - taf-secty taf-no-secty taf-perf taf-perf-no-secty -
 - ds-bacnet ds-camera ds-grove ds-modbus ds-mqtt ds-random ds-rest ds-snmp ds-virtual -
 - asc-http asc-mqtt -
 - modbus-sim ui ui-only -
endef
export OPTIONS



ARGS:=$(wordlist 2,$(words $(MAKECMDGOALS)),$(MAKECMDGOALS))
$(eval $(ARGS):;@:)

# Order of processing these arguments is important so the overrides in the 'add' compose files occur properly.
ifeq (dev, $(filter dev,$(ARGS)))
	CORE_EDGEX_REPOSITORY=edgexfoundry
	DEV:=-dev
	# matches the way that edgex-go to versioning the docker images for `make docker`
	CORE_EDGEX_VERSION:="0.0.0"
endif
ifeq (app-dev, $(filter app-dev,$(ARGS)))
	APP_SVC_REPOSITORY=edgexfoundry
	APP_SVC_DEV:=-dev
	APP_SERVICE_VERSION:="0.0.0"
endif
ifeq (arm64, $(filter arm64,$(ARGS)))
	ARCH:=-arm64
	KONG_UBUNTU:=-ubuntu
endif

# Add Device Services
ifeq (ds-bacnet, $(filter ds-bacnet,$(ARGS)))
	COMPOSE_FILES:=$(COMPOSE_FILES) -f add-device-bacnet.yml
	ifneq (no-secty, $(filter no-secty,$(ARGS)))
		ifeq ($(TOKEN_LIST),"")
			TOKEN_LIST:=device-bacnet
		else
			TOKEN_LIST:=$(TOKEN_LIST),device-bacnet
		endif
		ifeq ($(KNOWN_SECRETS_LIST),"")
			KNOWN_SECRETS_LIST:=redisdb[device-bacnet]
		else
			KNOWN_SECRETS_LIST:=$(KNOWN_SECRETS_LIST),redisdb[device-bacnet]
		endif
		extension_file:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh device-bacnet device-bacnet startup.sh "")
		COMPOSE_FILES:=$(COMPOSE_FILES) -f $(extension_file)
	endif
endif
ifeq (ds-camera, $(filter ds-camera,$(ARGS)))
	COMPOSE_FILES:=$(COMPOSE_FILES) -f add-device-camera.yml
	ifneq (no-secty, $(filter no-secty,$(ARGS)))
		ifeq ($(TOKEN_LIST),"")
			TOKEN_LIST:=device-camera-go
		else
			TOKEN_LIST:=$(TOKEN_LIST),device-camera-go
		endif
		ifeq ($(KNOWN_SECRETS_LIST),"")
			KNOWN_SECRETS_LIST:=redisdb[device-camera-go]
		else
			KNOWN_SECRETS_LIST:=$(KNOWN_SECRETS_LIST),redisdb[device-camera-go]
		endif
		extension_file:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh device-camera device-camera-go)
		COMPOSE_FILES:=$(COMPOSE_FILES) -f $(extension_file)
	endif
endif
ifeq (ds-grove, $(filter ds-grove,$(ARGS)))
    DS_GROVE:=true
	COMPOSE_FILES:=$(COMPOSE_FILES) -f add-device-grove.yml
	ifneq (no-secty, $(filter no-secty,$(ARGS)))
		ifeq ($(TOKEN_LIST),"")
			TOKEN_LIST:=device-grove
		else
			TOKEN_LIST:=$(TOKEN_LIST),device-grove
		endif
		ifeq ($(KNOWN_SECRETS_LIST),"")
			KNOWN_SECRETS_LIST:=redisdb[device-grove]
		else
			KNOWN_SECRETS_LIST:=$(KNOWN_SECRETS_LIST),redisdb[device-grove]
		endif
		extension_file:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh device-grove)
		COMPOSE_FILES:=$(COMPOSE_FILES) -f $(extension_file)
	endif
endif
ifeq (ds-modbus, $(filter ds-modbus,$(ARGS)))
	COMPOSE_FILES:=$(COMPOSE_FILES) -f add-device-modbus.yml
	ifneq (no-secty, $(filter no-secty,$(ARGS)))
		ifeq ($(TOKEN_LIST),"")
			TOKEN_LIST:=edgex-device-modbus
		else
			TOKEN_LIST:=$(TOKEN_LIST),edgex-device-modbus
		endif
		ifeq ($(KNOWN_SECRETS_LIST),"")
			KNOWN_SECRETS_LIST:=redisdb[edgex-device-modbus]
		else
			KNOWN_SECRETS_LIST:=$(KNOWN_SECRETS_LIST),redisdb[edgex-device-modbus]
		endif
		extension_file:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh device-modbus edgex-device-modbus device-modbus)
		COMPOSE_FILES:=$(COMPOSE_FILES) -f $(extension_file)
	endif
endif
ifeq (ds-mqtt, $(filter ds-mqtt,$(ARGS)))
	COMPOSE_FILES:=$(COMPOSE_FILES) -f add-device-mqtt.yml
	ifneq (no-secty, $(filter no-secty,$(ARGS)))
		ifeq ($(TOKEN_LIST),"")
			TOKEN_LIST:=edgex-device-mqtt
		else
			TOKEN_LIST:=$(TOKEN_LIST),edgex-device-mqtt
		endif
		ifeq ($(KNOWN_SECRETS_LIST),"")
			KNOWN_SECRETS_LIST:=redisdb[edgex-device-mqtt]
		else
			KNOWN_SECRETS_LIST:=$(KNOWN_SECRETS_LIST),redisdb[edgex-device-mqtt]
		endif
		extension_file:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh device-mqtt edgex-device-mqtt device-mqtt)
		COMPOSE_FILES:=$(COMPOSE_FILES) -f $(extension_file)
	endif
endif
ifeq (ds-random, $(filter ds-random,$(ARGS)))
	COMPOSE_FILES:=$(COMPOSE_FILES) -f add-device-random.yml
	ifneq (no-secty, $(filter no-secty,$(ARGS)))
		ifeq ($(TOKEN_LIST),"")
			TOKEN_LIST:=device-random
		else
			TOKEN_LIST:=$(TOKEN_LIST),device-random
		endif
		ifeq ($(KNOWN_SECRETS_LIST),"")
			KNOWN_SECRETS_LIST:=redisdb[device-random]
		else
			KNOWN_SECRETS_LIST:=$(KNOWN_SECRETS_LIST),redisdb[device-random]
		endif
		extension_file:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh device-random)
		COMPOSE_FILES:=$(COMPOSE_FILES) -f $(extension_file)
	endif
endif
ifeq (ds-rest, $(filter ds-rest,$(ARGS)))
	COMPOSE_FILES:=$(COMPOSE_FILES) -f add-device-rest.yml
	ifneq (no-secty, $(filter no-secty,$(ARGS)))
		# TODO: Remove setting token list once device rest's service key has been changed
		#       to 'device-rest' or edgex-go has been update to use 'edgex-device-rest'.
		ifeq ($(TOKEN_LIST),"")
			TOKEN_LIST:=edgex-device-rest
		else
			TOKEN_LIST:=$(TOKEN_LIST),edgex-device-rest
		endif
		ifeq ($(KNOWN_SECRETS_LIST),"")
			KNOWN_SECRETS_LIST:=redisdb[edgex-device-rest]
		else
			KNOWN_SECRETS_LIST:=$(KNOWN_SECRETS_LIST),redisdb[edgex-device-rest]
		endif
		extension_file:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh device-rest edgex-device-rest device-rest-go)
		COMPOSE_FILES:=$(COMPOSE_FILES) -f $(extension_file)
	endif
endif
ifeq (ds-snmp, $(filter ds-snmp,$(ARGS)))
	COMPOSE_FILES:=$(COMPOSE_FILES) -f add-device-snmp.yml
	ifneq (no-secty, $(filter no-secty,$(ARGS)))
		ifeq ($(TOKEN_LIST),"")
			TOKEN_LIST:=device-snmp
		else
			TOKEN_LIST:=$(TOKEN_LIST),device-snmp
		endif
		ifeq ($(KNOWN_SECRETS_LIST),"")
			KNOWN_SECRETS_LIST:=redisdb[device-snmp]
		else
			KNOWN_SECRETS_LIST:=$(KNOWN_SECRETS_LIST),redisdb[device-snmp]
		endif
		extension_file:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh device-snmp device-snmp-go device-snmp-go)
		COMPOSE_FILES:=$(COMPOSE_FILES) -f $(extension_file)
	endif
endif
ifeq (ds-virtual, $(filter ds-virtual,$(ARGS)))
	COMPOSE_FILES:=$(COMPOSE_FILES) -f add-device-virtual.yml
	ifneq (no-secty, $(filter no-secty,$(ARGS)))
		# Device-virtual's token is created by default.
		ifeq ($(KNOWN_SECRETS_LIST),"")
			KNOWN_SECRETS_LIST:=redisdb[device-virtual]
		else
			KNOWN_SECRETS_LIST:=$(KNOWN_SECRETS_LIST),redisdb[device-virtual]
		endif
		extension_file:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh device-virtual)
		COMPOSE_FILES:=$(COMPOSE_FILES) -f $(extension_file)
	endif
endif

# Add ModBus Simulator
ifeq (modbus-sim, $(filter modbus-sim,$(ARGS)))
	COMPOSE_FILES:=$(COMPOSE_FILES) -f add-modbus-simulator.yml
endif

# Add Application Services
ifeq (asc-http, $(filter asc-http,$(ARGS)))
	COMPOSE_FILES:=$(COMPOSE_FILES) -f add-asc-http-export.yml
	ifneq (no-secty, $(filter no-secty,$(ARGS)))
		ifeq ($(TOKEN_LIST),"")
			TOKEN_LIST:=app-http-export
		else
			TOKEN_LIST:=$(TOKEN_LIST),app-http-export
		endif
		ifeq ($(KNOWN_SECRETS_LIST),"")
			KNOWN_SECRETS_LIST:=redisdb[app-http-export]
		else
			KNOWN_SECRETS_LIST:=$(KNOWN_SECRETS_LIST),redisdb[app-http-export]
		endif
		# when no security mode (no-secty) not explicitly specified,
		# then we also need to add the secure version on top of base yml by default.
		extension_file:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh app-service-http-export \
			app-http-export app-service-configurable)
		COMPOSE_FILES:=$(COMPOSE_FILES) -f $(extension_file)
	endif
endif
ifeq (asc-mqtt, $(filter asc-mqtt,$(ARGS)))
	COMPOSE_FILES:=$(COMPOSE_FILES) -f add-asc-mqtt-export.yml
	ifneq (no-secty, $(filter no-secty,$(ARGS)))
		ifeq ($(TOKEN_LIST),"")
			TOKEN_LIST:=app-mqtt-export
		else
			TOKEN_LIST:=$(TOKEN_LIST),app-mqtt-export
		endif
		ifeq ($(KNOWN_SECRETS_LIST),"")
			KNOWN_SECRETS_LIST:=redisdb[app-mqtt-export]
		else
			KNOWN_SECRETS_LIST:=$(KNOWN_SECRETS_LIST),redisdb[app-mqtt-export]
		endif
		# when no security mode (no-secty) not explicitly specified,
		# then we also need to add the secure version on top of base yml by default.
		extension_file:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh app-service-mqtt-export \
			app-mqtt-export app-service-configurable)
		COMPOSE_FILES:=$(COMPOSE_FILES) -f $(extension_file)
	endif
endif

# Add a MQTT Broker
ifeq (mqtt-broker, $(filter mqtt-broker,$(ARGS)))
	COMPOSE_FILES:=$(COMPOSE_FILES) -f add-mqtt-broker.yml
endif

# Add switch to use MQTT Message Bus
ifeq (mqtt-bus, $(filter mqtt-bus,$(ARGS)))
	COMPOSE_FILES:=$(COMPOSE_FILES) -f add-mqtt-messagebus.yml -f add-mqtt-broker.yml
	BUS:=-mqtt
endif

# Add switch to use ZMQ Message Bus
ifeq (zmq-bus, $(filter zmq-bus,$(ARGS)))
	COMPOSE_FILES:=$(COMPOSE_FILES) -f add-zmq-messagebus.yml
	BUS:=-zmq
endif

# Add Security services and settings
ifeq (no-secty, $(filter no-secty,$(ARGS)))
	NO_SECURITY:=-no-secty
else
	COMPOSE_FILES:=$(COMPOSE_FILES) -f add-security.yml
endif

ifeq (ui, $(filter ui,$(ARGS)))
	COMPOSE_FILES:=$(COMPOSE_FILES) -f add-ui.yml
endif

ifeq (ui-only, $(filter ui-only,$(ARGS)))
	COMPOSE_FILES:=-f stand-alone-ui.yml -f add-ui.yml
	UI:=-ui
endif

# Build compose for TAF secure testing (ignore all other compose file options)
ifeq (taf-secty, $(filter taf-secty,$(ARGS)))
	# TODO: Remove Device Rest from token list once device rest's service key has been changed
	#       to 'device-rest' or edgex-go has been update to use 'edgex-device-rest'.
	TOKEN_LIST:=app-http-export,app-mqtt-export,app-functional-tests,scalability-test-mqtt-export,edgex-device-modbus,edgex-device-rest
	# Note that the services in this list should be separated by ';', but that causes issues with build scripts, so
	# have to list them individually.
	KNOWN_SECRETS_LIST:=redisdb[app-rules-engine],redisdb[app-http-export],redisdb[app-mqtt-export],redisdb[scalability-test-mqtt-export],redisdb[edgex-device-modbus],redisdb[edgex-device-rest]
	COMPOSE_FILES:= \
		-f docker-compose-base.yml \
		-f add-security.yml \
		-f add-taf-app-services.yml \
		-f add-taf-app-services-secure.yml \
		-f add-asc-http-export.yml \
		-f add-asc-mqtt-export.yml \
		-f add-device-virtual.yml \
		-f add-device-rest.yml \
		-f add-device-modbus.yml \
		-f add-taf-device-services-mods.yml \
		-f add-mqtt-broker.yml \
		-f add-modbus-simulator.yml
	asc_http_export_ext:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh app-service-http-export \
		app-http-export app-service-configurable)
	asc_mqtt_export_ext:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh app-service-mqtt-export \
		app-mqtt-export app-service-configurable)
	# TODO: Adjust Device Rest service key name once device rest's service key has been changed
	#       to 'device-rest' or remove this comment if edgex-go has been update to use 'edgex-device-rest'.
	ds_rest_ext:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh device-rest edgex-device-rest device-rest-go)
	# taf has its special place holder from taf-device-services-mods and thus we need to keep it
	# and extend security related things on top of it
	ds_virtual_ext:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh device-virtual \
		device-virtual device-virtual ' --registry --confdir=CONF_DIR_PLACE_HOLDER')
	ds_modbus_ext:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh device-modbus \
		edgex-device-modbus device-modbus ' --registry --confdir=CONF_DIR_PLACE_HOLDER')
	COMPOSE_FILES:=$(COMPOSE_FILES) -f $(asc_http_export_ext) -f $(asc_mqtt_export_ext)
	COMPOSE_FILES:=$(COMPOSE_FILES) -f $(ds_virtual_ext) -f $(ds_rest_ext) -f $(ds_modbus_ext)
else
	# Build compose for TAF non-secure testing (ignore all other compose file options)
    ifeq (taf-no-secty, $(filter taf-no-secty,$(ARGS)))
		COMPOSE_FILES:= \
			-f docker-compose-base.yml \
			-f add-taf-app-services.yml \
			-f add-asc-http-export.yml \
			-f add-asc-mqtt-export.yml \
			-f add-device-virtual.yml \
			-f add-device-rest.yml \
			-f add-device-modbus.yml \
			-f add-taf-device-services-mods.yml \
			-f add-mqtt-broker.yml \
			-f add-modbus-simulator.yml
        NO_SECURITY:=-no-secty
    else
    	# Build compose for TAF secure performance testing (ignore all other compose file options)
    	ifeq (taf-perf, $(filter taf-perf,$(ARGS)))
    		# TODO: Remove Device Rest from token list once device rest's service key has been changed
        	#       to 'device-rest' or edgex-go has been update to use 'edgex-device-rest'.
			TOKEN_LIST:=app-http-export,app-mqtt-export,app-functional-tests,scalability-test-mqtt-export,edgex-device-rest
			# Note that the services in this list should be separated by ';', but that causes issues with build scripts, so
			# have to list them individually.
			KNOWN_SECRETS_LIST:=redisdb[app-rules-engine],redisdb[app-http-export],redisdb[app-mqtt-export],redisdb[scalability-test-mqtt-export],redisdb[edgex-device-rest]

    		COMPOSE_FILES:= \
    			-f docker-compose-base.yml \
    			-f add-security.yml \
    			-f add-asc-mqtt-export.yml \
    			-f add-device-virtual.yml \
    			-f add-device-rest.yml \
    			-f add-mqtt-broker.yml
			asc_mqtt_export_ext:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh app-service-mqtt-export \
				app-mqtt-export app-service-configurable)
			ds_virtual_ext:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh device-virtual)
			# TODO: Adjust Device Rest service key name once device rest's service key has been changed
			#       to 'device-rest' or remove this comment if edgex-go has been update to use 'edgex-device-rest'.
			ds_rest_ext:= $(shell GEN_EXT_DIR="$(GEN_EXT_DIR)" ./gen_secure_compose_ext.sh device-rest edgex-device-rest device-rest-go)
			COMPOSE_FILES:=$(COMPOSE_FILES) -f $(asc_mqtt_export_ext) -f $(ds_virtual_ext) -f $(ds_rest_ext)
    	else
			# Build compose for TAF non-secure performance testing (ignore all other compose file options)
			ifeq (taf-perf-no-secty, $(filter taf-perf-no-secty,$(ARGS)))
				COMPOSE_FILES:= \
					-f docker-compose-base.yml \
					-f add-asc-mqtt-export.yml \
					-f add-device-virtual.yml \
					-f add-device-rest.yml \
					-f add-mqtt-broker.yml
				NO_SECURITY:=-no-secty
			endif
		endif
	endif
endif

SERVICES:=$(filter-out ${OPTIONS},$(ARGS))

ifeq (true, $(DS_GROVE))
 ifneq (-arm64, $(ARCH))
	 GROVE_CHECK:="ds-grove option only valid with arm64 option"
	 COMPOSE_FILES:=""
 endif
endif

help:
	echo "See README.md in this folder"

portainer:
	make -C ${RELEASE_FOLDER} portainer

portainer-down:
	make -C ${RELEASE_FOLDER} portainer-down

build:
	make compose ds-rest ds-virtual
	make compose ds-rest ds-virtual arm64
	make compose ds-rest ds-virtual no-secty
	make compose ds-rest ds-virtual no-secty arm64
	make compose ui-only
	make compose ui-only arm64
	make taf-compose taf-secty
	make taf-compose taf-no-secty
	make taf-compose taf-secty arm64
	make taf-compose taf-no-secty arm64
	make taf-compose-perf taf-perf
	make taf-compose-perf taf-perf-no-secty
	make taf-compose-perf taf-perf arm64
	make taf-compose-perf taf-perf-no-secty arm64

compose: gen
	cat gen-header docker-compose.yml > $(RELEASE_FOLDER)/docker-compose$(UI)$(NO_SECURITY)$(BUS)$(ARCH).yml

taf-compose: gen
	cat gen-header docker-compose.yml > $(RELEASE_FOLDER)/taf/docker-compose-taf$(NO_SECURITY)$(BUS)$(ARCH).yml

taf-compose-perf: gen
	cat gen-header docker-compose.yml > $(RELEASE_FOLDER)/taf/docker-compose-taf-perf$(NO_SECURITY)$(BUS)$(ARCH).yml

run: gen
	docker-compose -p edgex up -d $(SERVICES)

pull: gen
	docker-compose pull $(SERVICES)

gen:
	$(GROVE_CHECK) \
	DEV=$(DEV) \
	APP_SVC_DEV=$(APP_SVC_DEV) \
	CORE_EDGEX_REPOSITORY=$(CORE_EDGEX_REPOSITORY) \
	CORE_EDGEX_VERSION=$(CORE_EDGEX_VERSION) \
	APP_SVC_REPOSITORY=$(APP_SVC_REPOSITORY) \
	APP_SERVICE_VERSION=$(APP_SERVICE_VERSION) \
	ARCH=$(ARCH) \
	TOKEN_LIST=$(TOKEN_LIST) \
	KNOWN_SECRETS_LIST=$(KNOWN_SECRETS_LIST) \
	GEN_EXT_DIR=$(GEN_EXT_DIR) \
	docker-compose -p edgex $(COMPOSE_FILES) config > docker-compose.yml
	rm -rf ./$(GEN_EXT_DIR)

get-token:
	DEV=$(DEV) \
	ARCH=$(ARCH) \
	sh ./get-api-gateway-token.sh

upload-tls-cert:
	DEV=$(DEV) \
	ARCH=$(ARCH) \
	sh ./upload-api-gateway-cert.sh

get-consul-acl-token:
	DEV=$(DEV) \
	ARCH=$(ARCH) \
	sh ./get-consul-acl-token.sh

ui-down:
	ARCH= \
	docker-compose -p edgex -f stand-alone-ui.yml -f add-ui.yml down

down: ui-down
	DEV= \
	APP_SVC_DEV= \
	ARCH= \
	TOKEN_LIST= \
	KNOWN_SECRETS_LIST= \
	docker-compose -p edgex \
		-f docker-compose-base.yml \
		-f add-device-bacnet.yml \
		-f add-device-camera.yml \
		-f add-device-grove.yml \
		-f add-device-modbus.yml \
		-f add-device-mqtt.yml \
		-f add-device-random.yml \
		-f add-device-rest.yml \
		-f add-device-snmp.yml \
		-f add-device-virtual.yml \
		-f add-asc-http-export.yml \
		-f add-asc-mqtt-export.yml \
		-f add-modbus-simulator.yml \
		-f add-mqtt-broker.yml \
		-f add-mqtt-messagebus.yml \
		-f add-security.yml \
		down

clean: down
	-docker rm $$(docker ps --filter "network=edgex_edgex-network" --filter "network=edgex_default" -aq) 2> /dev/null
	docker volume prune -f && \
	docker network prune -f
