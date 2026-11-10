# 03. Requisitos do Software e Histórias de Usuário

Este documento apresenta a reconstrução parcial dos requisitos do **Font Manager**, extraídos com base no código-fonte, na documentação oficial e na interface gráfica do projeto original (`FontManager/font-manager`), além das histórias de usuário e cenários de teste associados.

## 1. Requisitos Funcionais (RF)

**RF01:** O sistema deve permitir a instalação de arquivos de fontes (`.ttf`, `.otf`) para o usuário atual ou no diretório do sistema.
* **Evidência no projeto:** Módulos de manipulação de arquivos e chamadas de sistema na pasta `src/lib/` que gerenciam a cópia para `~/.local/share/fonts` ou `/usr/share/fonts`. 

**RF02:** O sistema deve permitir a criação e o gerenciamento de coleções/grupos de fontes personalizados pelo usuário.
* **Evidência no projeto:** Interface de gerenciamento de coleções (`FontManager.Folder`, `FontManager.Collection`) e modelos de dados definidos em arquivos `.vala` na estrutura do projeto.

**RF03:** O sistema deve fornecer busca e filtragem dinâmica de fontes por nome, família e categoria tipográfica.
* **Evidência no projeto:** Componente de busca (`Gtk.SearchEntry`) integrado à visualização principal da lista de fontes na interface (`src/ui/`).

**RF04:** O sistema deve permitir a visualização de amostras de pré-visualização (preview) de texto personalizáveis para qualquer fonte selecionada.
* **Evidência no projeto:** Integração do componente de renderização tipográfica Pango (`Pango.Layout`) nos widgets de pré-visualização.

## 2. Requisitos Não Funcionais (RNF)

**RNF01:** Desempenho
* **Descrição do Requisito:** A interface gráfica deve responder sem travamentos (mantendo a UI fluida) ao varrer e carregar diretórios com mais de 500 arquivos de fontes.
* **Evidência no projeto:** Utilização de chamadas assíncronas e gerenciamento de threads via GLib/GTask na rotina de varredura de arquivos tipográficos. 

**RNF02:** Compatibilidade
* **Descrição do Requisito:** O software deve ser compatível e integrar-se nativamente ao ecossistema de desktop Linux baseado no ambiente GNOME / GTK.
* **Evidência no projeto:** Arquivos de configuração de compilação `meson.build` especificando dependências de `gtk+-3.0` / `gtk4`, `glib-2.0` e `pango`.

**RNF03:** Usabilidade
* **Descrição do Requisito:** O sistema deve oferecer suporte a atalhos de teclado e interface adaptativa seguindo as diretrizes de HIG (Human Interface Guidelines) do GNOME.
* **Evidência no projeto:** Definições de atalhos e acoplamento aos padrões visuais e layouts XML/UI do GNOME Builder e Libadwaita/GTK. 

**RNF04:** Manutenibilidade
* **Descrição do Requisito:** O código-fonte deve ser modularizado utilizando orientação a objetos em Vala, separando a lógica de negócio da interface do usuário.
* **Evidência no projeto:** Estrutura de diretórios organizada em `src/lib/` (lógica de backend/fontes) e `src/ui/` (interfaces e widgets GTK).

## 3. Histórias de Usuário e Cenários de Teste (BDD)

### História 1: Filtragem Rápida de Fontes por Busca

*Como* um designer gráfico, *quero* filtrar as fontes digitando seu nome no campo de busca, *para* localizar rapidamente uma tipografia específica sem precisar percorrer manualmente toda a lista do sistema.

* Cenário 1: Busca por nome existente
  * *Dado que* o usuário está na tela principal com todas as fontes carregadas,
  * *Quando* ele digita "Roboto" no campo de busca textual,
  * *Então* a lista de fontes deve ser atualizada para exibir apenas as fontes cujo nome contenha o termo "Roboto".

* Cenário 2: Busca por termo sem correspondência
  * *Dado que* o usuário está na tela principal do sistema,
  * *Quando* ele digita "FonteInexistenteXYZ" no campo de busca,
  * *Então* a lista deve ficar vazia e exibir uma mensagem informando que nenhuma fonte foi encontrada.


### História 2: Criação de Coleções de Fontes

*Como* um desenvolvedor front-end, *quero* criar uma coleção customizada chamada "Monospace", *para* agrupar apenas as fontes adequadas para escrita de código e separá-las de fontes ornamentais.

* Cenário 1: Criação com sucesso de uma nova coleção
  * *Dado que* o usuário está no menu de gerenciamento de coleções,
  * *Quando* ele clica no botão de adicionar nova coleção e digita o nome "Monospace",
  * *Então* uma nova coleção vazia chamada "Monospace" deve ser exibida na barra lateral.

* Cenário 2: Adição de fontes a uma coleção existente
  * *Dado que* a coleção "Monospace" já foi criada e está visível na barra lateral,
  * *Quando* o usuário seleciona a fonte "Fira Code" e a associa a essa coleção,
  * *Então* a fonte "Fira Code" deve figurar dentro do grupo "Monospace" ao selecionar a coleção.


### História 3: Instalação de Novos Arquivos de Fonte

*Como* um criador de conteúdo, *quero* instalar um arquivo `.ttf` baixado da internet diretamente pela interface do aplicativo, *para* disponibilizar a nova fonte em editores de imagem e texto do meu sistema.

* Cenário 1: Instalação bem-sucedida de um arquivo .ttf válido
  * *Dado que* o usuário possui um arquivo `CustomFont.ttf` válido em seu diretório de Downloads,
  * *Quando* ele seleciona a opção "Instalar Fonte" e escolhe o arquivo `CustomFont.ttf`,
  * *Então* o sistema deve copiar o arquivo para o diretório de fontes do usuário e atualizar a lista principal exibindo a nova fonte.

* Cenário 2: Tentativa de instalação de arquivo corrompido
  * *Dado que* o usuário seleciona um arquivo com extensão `.ttf` mas de conteúdo inválido/corrompido,
  * *Quando* ele confirma a tentativa de instalação,
  * *Então* o sistema deve exibir uma mensagem de erro indicando que o arquivo é inválido e interromper o processo sem alterar o sistema.

### História 4: Personalização da Amostra de Texto no Preview

*Como* um tipógrafo, *quero* alterar a frase de teste exibida na área de pré-visualização, *para* comparar o comportamento dos caracteres da fonte selecionada com um texto específico do meu projeto.

* Cenário 1: Alteração da frase de preview
  * *Dado que* o usuário selecionou uma fonte na lista e visualiza o preview com o texto padrão ("O rápido cachorro bronzeado..."),
  * *Quando* ele insere o texto "DESIGN 2026" no campo de amostra de pré-visualização,
  * *Então* a área de renderização deve atualizar imediatamente e desenhar a frase "DESIGN 2026" utilizando a fonte selecionada.

* Cenário 2: Ajuste no tamanho do fonte no preview
  * *Dado que* o usuário está visualizando a amostragem de texto da fonte,
  * *Quando* ele altera o seletor de tamanho de fonte de `12pt` para `48pt`,
  * *Então* a renderização do texto na tela deve redimensionar para a escala `48pt` sem distorções visuais.