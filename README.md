
# 📘 Documentação das Pipelines - MES Gusa

Este repositório contém as definições de pipelines de CI/CD do projeto **MES Gusa**, utilizando **Azure DevOps** com templates reutilizáveis para etapas de build, teste, publicação e deploy.

---

## 📂 Estrutura dos Arquivos

```
.
├── azure-pipelines.yml         # Pipeline principal (CI/CD)
├── steps-build.yaml            # Template com as etapas de build e publicação de artefatos
├── steps-deploy.yaml           # Template com as etapas de deploy da aplicação
```

---

## 🔄 CI - Integração Contínua (Build & Publish)

O pipeline de **CI** realiza as seguintes ações:

- Restaura pacotes NuGet
- Gera versão do projeto com GitVersion
- Compila a solução
- Executa testes automatizados
- Copia e publica os artefatos `WEB` e `WS`

### 📄 Arquivo: `steps-build.yaml`

Responsável por todo o processo de build, teste e publicação:

```yaml
- task: NuGetCommand@2
- task: gittools.gitversion.gitversion-task.GitVersion@5
- task: VSBuild@1
- task: VSTest@2
- task: CopyFiles@2           # Copia os arquivos da aplicação web
- task: CopyFiles@2           # Copia arquivos de backend (WS)
- task: PublishBuildArtifacts@1
```

Os artefatos gerados são armazenados nas pastas:

- `$(Build.ArtifactStagingDirectory)/WEB`
- `$(Build.ArtifactStagingDirectory)/WS`

---

## 🚀 CD - Entrega Contínua (Deploy)

O pipeline de **CD** é responsável por:

- Baixar os artefatos gerados no CI
- Mapear a unidade de rede
- Publicar arquivos nas pastas específicas (IIS)
- Desmapear a unidade de rede

### 📄 Arquivo: `steps-deploy.yaml`

```yaml
- template: download/files.yaml@pipelines-templates      # Baixa artefatos WEB e WS
- template: mapeamento/create.yaml@pipelines-templates   # Mapeia unidade de rede
- template: publish/publish-iis.yaml@pipelines-templates # Publica no IIS
- template: mapeamento/remove.yaml@pipelines-templates   # Remove mapeamento
```

> A publicação ocorre apenas para a branch `master`.

---

## ⚙️ Pipeline Principal (`azure-pipelines.yml`)

Define os estágios CI e CD:

```yaml
stages:
  - stage: BuildAndPush
    jobs:
      - job: BuildAndPublishProd
        steps:
          - template: steps-build.yaml

  - stage: Deploy
    dependsOn: BuildAndPush
    jobs:
      - deployment: DeployMESGusa
        environment: 'Deploy'
        strategy:
          runOnce:
            deploy:
              steps:
                - template: steps-deploy.yaml
```

---

## 🔐 Variáveis Utilizadas

| Nome                | Valor Padrão                      | Descrição                          |
|---------------------|-----------------------------------|-------------------------------------|
| `solution`          | `source/MES/**/*.sln`             | Caminho do arquivo de solução      |
| `configuration`     | `release`                         | Configuração da build              |
| `platform`          | `any cpu`                         | Plataforma da build                |
| `ArtifactNameWEB`   | `WEB`                             | Nome do artefato da aplicação web  |
| `ArtifactNameWS`    | `WS`                              | Nome do artefato dos serviços      |

---

## 📚 Repositório de Templates

O pipeline importa templates reutilizáveis do repositório externo:

```yaml
resources:
  repositories:
    - repository: pipelines-templates
      type: git
      name: 'Testes Gabriel/pipelines-templates'
      ref: 'refs/heads/master'
```

---

## ✅ Condições

- A execução do CI/CD é feita apenas quando a branch é `master`.
- O deploy ocorre somente se a build anterior for bem-sucedida.
- O uso de templates garante reutilização e padronização das etapas.

---

## 📎 Observações

- Certifique-se de que as máquinas de build e deploy tenham acesso ao caminho de rede mapeado.
- O publish via IIS é feito para os diretórios:
  - `n:\WebGusaGabriel` (Web)
  - `n:\WsGusaGabriel` (Serviços)

---

## ✨ Melhorias Futuras

- Implementar testes automatizados mais robustos
- Adicionar controle de qualidade com SonarCloud
- Parametrizar ambientes (Homologação, Produção)

---

Se tiver dúvidas, consulte os templates no repositório `pipelines-templates` ou fale com o responsável pelo pipeline.


---

## 👨‍💻 Autor

- GitHub: [@vilelaGa](https://github.com/vilelaGa)
- LinkedIn: [vilelaga](https://www.linkedin.com/in/vilelaga/)
