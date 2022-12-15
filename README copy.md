Referência: 
https://docs.google.com/document/d/1RGb6j9uDvak_3mLbHuKvwzflYBnkgt6yqq1hqnituzU/edit?pli=1#

Pressupõe-se já estar logado no OC, com um usuário com permissões de consulta ao projeto openshift-gitops

1 - Login no ArgoCD com usuário admin:
$ ARGOCD_ROUTE=$(oc -n openshift-gitops get route openshift-gitops-server -o jsonpath='{.spec.host}')
$ ADMIN_PASSWORD=$(oc get secret/openshift-gitops-cluster -n openshift-gitops --template='{{index .data "admin.password" | base64decode}}')
$ argocd login ${ARGOCD_ROUTE}:443 --insecure --username=admin --password=${ADMIN_PASSWORD}

2 - Adição do repositório Git ao ArgoCD
$ argocd repo add http://git-gogs-unsafe-gogs.apps-crc.testing/gogs-admin/gitops-pipelines-config.git --name gitops-namespaces-config --username gogs-admin --password admin

Utilizamos o usuário de leitura que criamos no Argo. Substituir xxxx pelo password do usuário.
Atenção ao ".git" ao final do repositório.

3 - Criação da aplicação

Deve-se fazer o clone da aplicação e entrar na pasta :
$ git clone http://git-gogs-unsafe-gogs.apps-crc.testing/gogs-admin/gitops-pipelines-config.git
$ cd gitops-namespaces-config


Segue abaixo comando de criação da aplicação no ArgoCD
$ argocd app create gitops-pipelines-config --dest-server https://kubernetes.default.svc --repo http://git-gogs-unsafe-gogs.apps-crc.testing/gogs-admin/gitops-pipelines-config.git --path . --sync-policy automated


Este passo só precisa ser feito uma vez. A partir deste momento, a aplicação já estará criada e qualquer alteração no Git será sincronizada com o Argo. 

Atualizar o password com token com permissão para movimentar a imagem entre namespaces (homologação para produção) 
```
pipelines/template/secret/registry-token.yaml
```