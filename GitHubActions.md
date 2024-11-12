No GitHub Actions, existem diversas variáveis de ambiente pré-definidas que são utilizadas para fornecer informações sobre a execução dos workflows. Aqui está uma lista com algumas das principais variáveis de ambiente, seguidas de exemplos:
1. GITHUB_WORKFLOW

    Descrição: Nome do workflow que está sendo executado.
    Exemplo: Se o workflow for chamado "CI/CD Pipeline", o valor da variável será:

```hcl
    GITHUB_WORKFLOW=CI/CD Pipeline
```

2. GITHUB_RUN_ID

    Descrição: ID único do fluxo de trabalho dentro do repositório. Permanece constante ao longo da execução.
    Exemplo:
```hcl
    GITHUB_RUN_ID=123456789
```

3. GITHUB_RUN_NUMBER

    Descrição: Número de execução do workflow, que aumenta a cada execução.
    Exemplo:
```hcl
    GITHUB_RUN_NUMBER=42
```

4. GITHUB_JOB

    Descrição: Nome do trabalho (job) que está sendo executado.
    Exemplo:
```hcl
    GITHUB_JOB=build-and-test
```

5. GITHUB_ACTION

    Descrição: Nome da ação em execução atualmente.
    Exemplo:
```hcl
    GITHUB_ACTION=my-custom-action
```

6. GITHUB_ACTOR

    Descrição: Nome do usuário que iniciou a execução (por exemplo, o autor de um pull request ou commit).
    Exemplo:
```hcl
    GITHUB_ACTOR=username123
```

7. GITHUB_REPOSITORY

    Descrição: Nome do repositório no formato owner/repo.
    Exemplo:
```hcl
    GITHUB_REPOSITORY=usuario/repositorio-exemplo
```

8. GITHUB_EVENT_NAME

    Descrição: Tipo de evento que acionou o workflow (ex.: push, pull_request).
    Exemplo:
```hcl
    GITHUB_EVENT_NAME=pull_request
```

9. GITHUB_EVENT_PATH

    Descrição: Caminho para o arquivo JSON que contém os dados do evento que acionou o workflow.
    Exemplo:
```hcl
    GITHUB_EVENT_PATH=/home/runner/work/_temp/_github_event.json
```

10. GITHUB_SHA

    Descrição: SHA do commit que acionou o workflow.
    Exemplo:
```hcl
    GITHUB_SHA=3a6f5b9c1d7e8c9f0b3f2e2a7a8b7e6c8d1e0f1a
```

11. GITHUB_REF

    Descrição: Referência da branch ou tag que acionou o workflow.
    Exemplo:
```hcl
    GITHUB_REF=refs/heads/main
```

12. GITHUB_HEAD_REF e GITHUB_BASE_REF

    Descrição: Variáveis relacionadas a pull requests. GITHUB_HEAD_REF é a branch de onde veio o PR e GITHUB_BASE_REF é a branch de destino.
    Exemplo:
```hcl
    GITHUB_HEAD_REF=feature/nova-funcionalidade
    GITHUB_BASE_REF=main
```

13. RUNNER_OS

    Descrição: Sistema operacional em que o runner está sendo executado.
    Exemplo:
```hcl
    RUNNER_OS=Linux
```

14. RUNNER_TEMP

    Descrição: Caminho para um diretório temporário no runner.
    Exemplo:
```hcl
    RUNNER_TEMP=/home/runner/work/_temp
```

15. RUNNER_TOOL_CACHE

    Descrição: Caminho para a pasta onde as ferramentas instaladas são armazenadas.
    Exemplo:
```hcl
    RUNNER_TOOL_CACHE=/opt/hostedtoolcache
```

Essas variáveis são automaticamente configuradas pelo GitHub Actions e podem ser utilizadas em comandos shell ou scripts para personalizar e automatizar tarefas.