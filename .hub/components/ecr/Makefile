.DEFAULT_GOAL := deploy

COMPONENT_NAME ?= ecr
DOMAIN_NAME    ?= dev.kubernetes.delivery

STATE_BUCKET ?= terraform.agilestacks.com
STATE_REGION ?= us-east-1

export AWS_DEFAULT_REGION ?= us-east-2

export TF_LOG      ?= info
export TF_DATA_DIR ?= .terraform/$(DOMAIN_NAME)-$(subst /,-,$(COMPONENT_NAME))
export TF_LOG_PATH ?= $(TF_DATA_DIR)/terraform.log

export TF_VAR_name := $(shell echo '$(COMPONENT_NAME)' | tr '[:upper:]' '[:lower:]' | sed -e 's/^[[:space:]]*//;s/[[:space:]]*$$//;s/[[:space:]]/_/g')

terraform   ?= terraform-v0.11
TF_CLI_ARGS ?= -no-color -input=false
TFPLAN      := $(TF_DATA_DIR)/terraform.tfplan

deploy: init import plan apply

$(TF_DATA_DIR):
	@rm -rf $@ && mkdir -p $@

init: $(TF_DATA_DIR)
	$(terraform) init -get=true -force-copy $(TF_CLI_ARGS) \
        -backend=true -reconfigure \
        -backend-config="bucket=$(STATE_BUCKET)" \
        -backend-config="region=$(STATE_REGION)" \
        -backend-config="key=$(DOMAIN_NAME)/ecr/$(subst /,-,$(COMPONENT_NAME))/terraform.tfstate" \
        -backend-config="profile=$(AWS_PROFILE)"

plan:
	$(terraform) plan $(TF_CLI_ARGS) -refresh=true -module-depth=-1 -out=$(TFPLAN)

refresh:
	$(terraform) refresh $(TF_CLI_ARGS)

apply:
	$(terraform) apply $(TF_CLI_ARGS) -auto-approve $(TFPLAN)

output:
	@$(terraform) output -no-color

undeploy: init import destroy apply

destroy: TF_CLI_ARGS:=-destroy $(TF_CLI_ARGS)
destroy: plan

import:
	-$(terraform) import $(TF_CLI_ARGS) module.ecr.aws_ecr_repository.main $(TF_VAR_name)

clean:
	rm -rf .terraform

-include ../Mk/phonies
