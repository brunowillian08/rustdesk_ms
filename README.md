# MSSuporte - Cliente de Acesso Remoto

Este repositório contém o código-fonte do **MSSuporte**, uma solução de acesso remoto customizada baseada no projeto open-source RustDesk. O cliente foi otimizado para distribuição ágil e implantação em ambientes de administração pública, focando em um executável portátil (Portable) de fácil execução.

## 🚀 Como Compilar (GitHub Actions)

A esteira de integração contínua (CI/CD) foi totalmente automatizada para contornar limitações de compilação cruzada (Flutter/Rust).

1. Acesse a aba **Actions** no GitHub.
2. Selecione o workflow **Build MS Suporte**.
3. Clique em **Run workflow** (utilizando a branch `main`).
4. Ao final do processo (aprox. 30-40 minutos), baixe o artefato `MSSuporte-portable-x64.exe` gerado na página de resumo da execução.

> **Nota Estratégica:** O projeto foi configurado para **NÃO** gerar o instalador `.msi`. Todo o fluxo está concentrado em gerar o executável Portable, que permite ao usuário final rodar a aplicação imediatamente ou instalá-la como Serviço do Windows clicando no botão "Instalar" da própria interface.

---

## 🛠️ Manifesto de Alterações (Changelog & Hacks)

Para garantir que o código compilasse sem o erro fatal de ponte (Bridge Panic - `Exit code 101`) e mantivesse a identidade visual do **MSSuporte**, aplicamos a tática do "Cavalo de Troia" na esteira de compilação:

### 1. A Mágica do CI/CD (`.github/workflows/compilar.yml`)
* **Preservação da Base:** Os arquivos raiz `Cargo.toml` e `CMakeLists.txt` foram mantidos com o nome original `rustdesk` para não quebrar a geração da ponte (bridge) entre o motor Rust e a interface Flutter.
* **Renomeação Dinâmica:** Inserimos scripts no workflow que interceptam o executável `rustdesk.exe` logo após a compilação base e o renomeiam fisicamente para `MSSuporte.exe` **antes** do empacotador (`generate.py`) entrar em ação.
* **Limpeza:** Remoção completa das dependências do MSBuild (WiX Toolset), economizando tempo de compilação.

### 2. Motor e Serviço do Windows (Rust)
* **Arquivo:** `libs/hbb_common/src/config.rs`
* **Alteração:** A constante `APP_NAME` foi alterada para `"MSSuporte"`. Isso garante que, quando o usuário instalar o portátil, a pasta em `C:\Program Files\` e o Serviço de inicialização do Windows sejam registrados com o nome correto, evitando o "Erro 2" (caminho não encontrado).

### 3. Interface de Usuário (Flutter)
* **Arquivo:** `flutter/lib/desktop/pages/desktop_home_page.dart`
* **Alterações de Layout:** 
  * Estrutura da coluna principal envolvida em um widget `Center`.
  * Eixos ajustados (`MainAxisAlignment.center` e `CrossAxisAlignment.center`) para flutuar o painel de conexão no meio exato da janela.
  * Margens engessadas (`EdgeInsets.only(left: 20)`) substituídas por simetria (`symmetric(horizontal: 20)`).
  * Textos de apoio centralizados nativamente com `TextAlign.center`.

---

## 📋 Próximos Passos (To-Do)

Caso haja necessidade de evoluções futuras no uso diário:

* **Homologação:** Testar a instalação do Serviço do Windows via botão "Instalar" do portátil em um ambiente limpo para confirmar o comportamento do UAC (Controle de Conta de Usuário).
* **Atualizações Base:** Quando houver necessidade de trazer atualizações do repositório oficial do RustDesk, deve-se tomar cuidado ao fazer *merge* nos arquivos `.github/workflows/` e `desktop_home_page.dart` para não sobrescrever o layout e a esteira customizada.
* **Servidor de ID/Relay:** Se futuramente for necessário apontar para um servidor próprio (HBB) em vez dos servidores públicos, as chaves e IPs devem ser fixados em `libs/hbb_common/src/config.rs`.
