# pihole-unbound

Este repositório contém a configuração para deploy automatizado do Unbound (DNS recursivo) e do Pi-hole (bloqueador de anúncios e rastreadores) utilizando Kubernetes e ArgoCD, proporcionando uma solução completa de DNS seguro, privado e com filtragem de conteúdo.

## Estrutura do Projeto

```
README.md
setup/
  project.yaml           # Definição do AppProject do ArgoCD
  repository.yaml        # Configuração do repositório Git para o ArgoCD
pihole/
  setup/
    application.yaml     # Definição da aplicação Pi-hole para o ArgoCD
    metallb-config.yaml  # Configuração do pool de IP do MetalLB para o serviço LoadBalancer do Pi-hole
    secret.yaml          # Senha/API do Pi-hole
  values/
    deployment.yaml      # Deployment do Pi-hole
    ingress.yaml         # Ingress para acesso web ao Pi-hole
    persistent-volume.yaml # PersistentVolume e PersistentVolumeClaim do Pi-hole
    service.yaml         # Services (LoadBalancer e ClusterIP) do Pi-hole, com anotação para MetalLB
unbound/
  setup/
    application.yaml     # Definição da aplicação Unbound para o ArgoCD
  values/
    configMap.yaml       # ConfigMap com a configuração do Unbound
    cronJob.yaml         # CronJob para atualização diária do root.hints
    deployment.yaml      # Deployment do Unbound
    namespace.yaml       # Namespace dedicado
    persistent-volume.yaml # PersistentVolume e PersistentVolumeClaim do Unbound
    service.yaml         # Service para expor o Unbound no cluster
```

## Pré-requisitos

- Cluster Kubernetes funcional
- ArgoCD instalado e configurado
- Acesso SSH ao repositório Git (chave privada configurada)
- [MetalLB](https://metallb.universe.tf/) instalado para LoadBalancer local (obrigatório para expor o DNS na rede local)

## Passos para Deploy

1. **Configure o acesso ao repositório:**

   Edite o arquivo [`setup/repository.yaml`](setup/repository.yaml) e insira sua chave SSH privada e o `repoURL` do seu fork ou repositório.

2. **Configure o projeto no ArgoCD:**

   Edite o arquivo [`setup/project.yaml`](setup/project.yaml) para garantir que o `repoURL` está correto.

3. **Configure o MetalLB:**

   - O arquivo [`pihole/setup/metallb-config.yaml`](pihole/setup/metallb-config.yaml) define o pool de IPs do MetalLB (exemplo: `192.168.0.3`) e a advertisement para o serviço LoadBalancer do Pi-hole.
   - Certifique-se de aplicar esta configuração no cluster antes de criar o serviço LoadBalancer.

4. **Configure as aplicações Pi-hole e Unbound:**

   - Edite [`pihole/setup/application.yaml`](pihole/setup/application.yaml) e [`unbound/setup/application.yaml`](unbound/setup/application.yaml) para ajustar o `repoURL` se necessário.
   - Certifique-se de que o namespace `pihole-unbound` será criado automaticamente (já configurado nos manifests).

5. **Adicione os manifests ao ArgoCD:**

   No ArgoCD, adicione o projeto e as aplicações utilizando os manifests acima.

   - O ArgoCD irá criar o namespace, volumes, configmaps, secrets, deployments, services, ingress e cronjob automaticamente.

6. **Personalize as configurações:**

   - Edite [`pihole/values/deployment.yaml`](pihole/values/deployment.yaml) para ajustar variáveis de ambiente do Pi-hole.
   - Edite [`pihole/setup/secret.yaml`](pihole/setup/secret.yaml) para definir a senha do painel web do Pi-hole.
   - Edite [`unbound/values/configMap.yaml`](unbound/values/configMap.yaml) para ajustar as opções do Unbound.

## Observações

- O serviço LoadBalancer do Pi-hole ([`pihole/values/service.yaml`](pihole/values/service.yaml)) está anotado com o pool do MetalLB (`metallb.io/address-pool: pool-local-dns`) e será exposto no IP definido em [`pihole/setup/metallb-config.yaml`](pihole/setup/metallb-config.yaml) (exemplo: `192.168.0.3`).
- O serviço ClusterIP do Pi-hole permite acesso ao painel web internamente e via Ingress.
- O Ingress [`pihole/values/ingress.yaml`](pihole/values/ingress.yaml) permite acesso ao painel web do Pi-hole pelo endereço `dns.homelab`.
- O volume persistente do Pi-hole é configurado para `/volumes/ssd/k3s/pihole` no host. Ajuste conforme necessário em [`pihole/values/persistent-volume.yaml`](pihole/values/persistent-volume.yaml).
- O volume persistente do Unbound é configurado para `/volumes/ssd/k3s/unbound` no host. Ajuste conforme necessário em [`unbound/values/persistent-volume.yaml`](unbound/values/persistent-volume.yaml).
- O serviço Unbound é exposto internamente no cluster nas portas 53/UDP e 53/TCP.
- O CronJob atualiza diariamente o arquivo `root.hints` do Unbound para manter a lista de root servers atualizada.

## Fluxo recomendado

1. O Pi-hole recebe requisições DNS dos clientes da rede.
2. O Pi-hole encaminha as requisições para o Unbound, que faz a resolução recursiva e segura.
3. O Unbound consulta diretamente os root servers, garantindo privacidade e independência de terceiros.

## Referências

- [Unbound DNS](https://www.nlnetlabs.nl/projects/unbound/about/)
- [Pi-hole](https://pi-hole.net/)
- [ArgoCD](https://argo-cd.readthedocs.io/en/stable/)
- [MetalLB](https://metallb.universe.tf/)

---

Mantenha seu repositório seguro: **NUNCA** compartilhe sua chave privada em repositórios públicos.

