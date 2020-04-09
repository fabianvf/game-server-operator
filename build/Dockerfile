FROM quay.io/operator-framework/ansible-operator:v0.16.0

COPY requirements.yml ${HOME}/requirements.yml
RUN ansible-galaxy collection install -r ${HOME}/requirements.yml \
 && chmod -R ug+rwx ${HOME}/.ansible

COPY roles/ ${HOME}/roles/
COPY reconcile.yml ${HOME}/reconcile.yml
COPY watches.yaml ${HOME}/watches.yaml
