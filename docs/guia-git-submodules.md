# Guia Rápido: Git Submodules para Desenvolvedores

Este documento é um guia prático para quem nunca trabalhou com **Git Submodules**. Siga os passos abaixo para entender e utilizar submódulos em projetos reais.

## 1. O que é um Submodule?
Um **submodule** é um repositório Git dentro de outro repositório Git. Ele permite que você inclua e gerencie dependências externas (outros projetos) mantendo o histórico e controle de versões separados.

## 2. Clonando o repositório com todos os submodules
Para clonar o projeto e já baixar todos os submodules, use:

```sh
git clone --recurse-submodules -j8 git@github.com:AelcioJozias/smartsensors.git
```

- O parâmetro `--recurse-submodules` garante que os submodules sejam clonados.
- O parâmetro `-j8` paraleliza o download (opcional, pode ajustar conforme seu hardware).

## 3. Corrigindo submodules vazios após clone normal
Se você clonou o repositório sem o parâmetro acima, as pastas dos submodules estarão vazias. Para corrigir:

```sh
git submodule update --init --recursive
```

## 4. Atualizando submodules no dia a dia
Para trazer as últimas mudanças do projeto principal **e** dos submodules:

```sh
git pull --recurse-submodules
```

## 5. Adicionando um novo submodule
Para adicionar um novo submodule ao projeto:

```sh
git submodule add <url-do-repositorio> <caminho/desejado>
```

**Exemplo real:**
```sh
git submodule add git@github.com:AelcioJozias/smart-sensors-novo-modulo.git microservices/novo-modulo
```

## 6. Dica: Criando alias para comandos frequentes
Você pode criar atalhos (alias) para facilitar o uso dos comandos no seu `.gitconfig`:

```ini
[alias]
    cloneall = !git clone --recurse-submodules -j8
    subupdate = submodule update --init --recursive
    pullall = pull --recurse-submodules
```

Assim, basta usar:
- `git cloneall <repo>`
- `git subupdate`
- `git pullall`

---

Siga este guia para evitar problemas e ganhar produtividade com submodules!

