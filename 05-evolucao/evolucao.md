# 5. Evolução proposta: renomear fontes diretamente no Font Manager

 Análise feita sobre o repositório oficial [FontManager/font-manager](https://github.com/FontManager/font-manager) e o código-fonte analisado (`lib/gtk/` e `lib/fontconfig/`).

---

## 5.1 Proposta

### Problema

O Font Manager permite inspecionar, ativar, desativar, instalar e remover fontes, mas **não oferece nenhum mecanismo para renomear uma fonte ou ajustar seu nome exibido** dentro da aplicação. Usuários frequentemente instalam arquivos tipográficos obtidos de terceiros com metadados malformados, nomes duplicados ou caracteres inválidos/de controle.

Hoje o usuário enfrenta duas limitações diretas na interface:

- **Painel de propriedades exclusivamente de leitura:** No widget `FontManagerPropertiesPage` (`lib/gtk/font-manager-properties-page.c`), todos os campos exibidos na grade (`psname`, `family`, `style`, `width`, `slant`, `weight`, `spacing`, `version`, `vendor`, `filetype`, `filesize`) são renderizados apenas como `GtkLabel` (ou `GtkLinkButton` no caso de `filesize`), sem qualquer controle de edição.
- **Conflitos e caracteres inválidos propagados para outros softwares:** Fontes com caracteres de controle no nome (como o caractere Unicode não imprimível `0x007F`, relatado na Issue #275) aparecem quebradas na listagem e falham ao serem selecionadas em ferramentas externas de design, como o Figma. Sem suporte na aplicação, o usuário precisa recorrer a ferramentas externas de edição de fontes ou manipular manualmente arquivos e regras do Fontconfig.

### Usuário afetado

Quem instala e organiza fontes locais no Linux e depende de identificação consistente entre o gerenciador e aplicativos de criação/edição:

- **Designers e criadores de conteúdo**, que instalam famílias tipográficas de fontes variadas e precisam corrigir nomes conflitantes ou com caracteres inválidos para utilizá-las no Figma, GIMP ou Inkscape.
- **Usuários finais e desenvolvedores**, que mantêm coleções locais de fontes e precisam diferenciar variantes ou corrigir metadados inconsistentes sem editar binários manualmente.

### Evidências

| Evidência | O que mostra |
| :--- | :--- |
| [Issue #275](https://github.com/FontManager/font-manager/issues/275) — *"Rename fonts inside font manager [FR]"* (aberta em 04/02/2022) | Solicitação direta da funcionalidade para evitar conflitos e corrigir nomes inválidos, relatando o caso real de uma fonte cujo nome continha o caractere `0x007F` e quebrava completamente dentro do Figma. A issue pede explicitamente ação via menu de contexto e edição no painel de propriedades. |
| Código de `lib/gtk/font-manager-properties-page.c` | A estrutura `_FontManagerPropertiesPage` e a função `construct_start_child()` instanciam apenas `GtkLabel` para exibir `family` e demais propriedades a partir de um `JsonObject`, sem `GtkEntry`, botões de ação ou sinais de alteração de nome. |
| Código de `lib/gtk/font-manager-preview-pane.c` | Em `font_manager_preview_pane_update_metadata()`, os metadados são carregados do SQLite (`SELECT * FROM Metadata ...`) ou via `font_manager_get_metadata()` e repassados de forma unidirecional para `font_manager_font_properties_page_update()`. |
| Código de `lib/fontconfig/font-manager-fontconfig.c` e `font-manager-alias.c` | O projeto já possui infraestrutura para ler `FC_FAMILY` / `FC_STYLE` e manipular elementos `<alias>` do Fontconfig (`FontManagerAliasElement`), mas ainda não expõe um fluxo de renomeação de fontes na camada GTK. |

### Solução proposta

Permitir que o usuário renomeie o nome de família (`family`) da fonte selecionada diretamente no **Painel de Propriedades (`Properties Pane`)**, com validação de entrada antes de confirmar a alteração:

- Na aba **Properties** do `FontManagerPreviewPane`, disponibilizar um controle para iniciar a edição do nome (`family`) por meio de um campo de texto (`GtkEntry`) acompanhado de ações de confirmação e cancelamento.
- Validar o nome informado antes de aceitar a mudança, bloqueando strings vazias, texto composto apenas por espaços, separadores de caminho (`/`, `\`) e caracteres de controle Unicode (como `0x007F`).
- Atualizar o estado local da propriedade `"family"` no `JsonObject` ativo e emitir o sinal `GObject` `"font-renamed"` para permitir integração com camadas superiores (como persistência via Fontconfig ou atualização da lista principal).

### Benefício esperado

- Permite corrigir rapidamente nomes de fontes inválidos ou conflitantes diretamente na interface onde o usuário inspeciona os metadados.
- Impede a inserção de caracteres de controle (como `0x007F`) e strings vazias por meio de validação imediata com feedback visual.
- Estabelece a interface e o contrato de sinais (`"font-renamed"`) necessários para evoluções futuras de persistência sem quebrar o funcionamento atual do painel.

### Prioridade

**Média.**

- **A favor de priorizar:** Resolve uma dor real e documentada na comunidade (Issue #275), possui escopo modular bem isolado nos widgets GTK (`lib/gtk/`) e reaproveita integralmente a stack nativa (GTK, GLib, JSON-GLib) sem exigir novas dependências no `meson.build`.
- **Contra:** Não corrige uma falha crítica de segurança ou travamento (*crash*), e uma solução definitiva completa exige decidir entre reescrever tabelas internas dos arquivos `.ttf`/`.otf` ou persistir regras de substituição no Fontconfig/SQLite.

---

## 5.2 MVP (Produto Mínimo Viável)

### O que será desenvolvido

1. **Modo de edição no painel de propriedades (`FontManagerPropertiesPage`):** Adição de controles de interface (`GtkEntry`, botões de iniciar, confirmar e cancelar edição, além de `GtkLabel` para mensagens de erro) para editar o nome da fonte (`family`).
2. **Validação de entrada (`is_valid_font_name`):** Bloqueio de strings vazias, strings contendo apenas espaços em branco, barras (`/`, `\`) e caracteres de controle (`g_unichar_iscntrl`), exibindo feedback visual ao usuário quando a entrada for inválida.
3. **Atualização local e sinal `GObject` (`font-renamed`):** Atualização do campo `"family"` no `JsonObject` de propriedades da sessão atual, re-renderização do painel (`update()`) e emissão do sinal `"font-renamed"` com o nome antigo e o novo nome validado.
4. **Sincronização de estado ao trocar fonte ou aba (`FontManagerPreviewPane`):** Exposição de `font_manager_font_properties_page_cancel_rename()` e chamada automática ao trocar de fonte (`font_manager_preview_pane_set_font`) ou alternar abas (`on_page_switch`), evitando que um modo de edição aberto vaze para outra fonte.
5. **Verificação de casos de teste lógicos e de borda:** Validação estruturada dos cenários de entrada vazia, espaços em branco, caractere de controle `0x007F`, barras, entrada válida com espaços nas bordas e cancelamento/troca de seleção.

### Valor entregue

Entrega um fluxo funcional e seguro de edição e validação de nome no painel de propriedades — atendendo à segunda metade da proposta da Issue #275 (*"It should also be editable inside the properties pane"*) — com baixo acoplamento e contrato pronto para receber persistência em etapas seguintes.

### Fora do MVP

| Fica de fora | Por quê |
| :--- | :--- |
| Edição binária das tabelas internas dos arquivos `.ttf` / `.otf` (tabela `name` OpenType) | Exige manipulação binária complexa de estruturas tipográficas ou bibliotecas externas especializadas, aumentando o risco de corrupção de arquivos de fonte neste protótipo. |
| Persistência definitiva em regras `<alias>` do Fontconfig ou no banco SQLite (`Metadata`) | Envolve escrita de XML de configuração do usuário, reindexação do cache do Fontconfig (`fc-cache`) e sincronização com o banco local; fica como evolução após validar o fluxo da UI. |
| Ação *"Rename"* no menu de contexto (botão direito) da lista principal | Depende de alterações nos modelos e menus da janela principal fora do recorte de `FontManagerPropertiesPage` e `FontManagerPreviewPane`. |
| Renomeação de fontes instaladas globalmente no sistema (`/usr/share/fonts`) e renomeação em lote | Fontes globais exigem privilégios administrativos (`root`/Polkit) e renomeação em lote aumenta significativamente o escopo de interface e regras de conflito. |

### Justificativa das escolhas

- Focar no `FontManagerPropertiesPage` ataca diretamente metade do pedido da Issue #275 em um componente coeso e já responsável por exibir os metadados da fonte selecionada.
- Implementar a validação com `g_unichar_iscntrl()` e `g_strstrip()` (GLib) resolve exatamente o problema de caracteres não imprimíveis como `0x007F` sem introduzir dependências externas no build Meson.
- Desacoplar a confirmação da persistência por meio do sinal `"font-renamed"` permite testar e revisar a interface e as regras de validação de forma independente antes de acoplar escrita em disco ou banco.

---

## 5.3 Análise de impacto

| Tipo | Parte afetada | Mudança | Impacto |
| :--- | :--- | :--- | :--- |
| **Interface: cabeçalho público** | `lib/gtk/font-manager-properties-page.h` | Declaração da função pública `font_manager_font_properties_page_cancel_rename()` | Baixo. Mantém compatibilidade total com os consumidores existentes da classe. |
| **Interface: painel de propriedades** | `lib/gtk/font-manager-properties-page.c` (`struct _FontManagerPropertiesPage`, `reset`, `class_init`, `init`) | Adição dos widgets de renomeação (`rename_entry`, botões e label de erro), sinal `"font-renamed"`, validação `is_valid_font_name()` e limpeza de estado no `reset()` | Médio. Concentra toda a lógica de interação e validação do MVP. |
| **Interface: painel de pré-visualização** | `lib/gtk/font-manager-preview-pane.c` (`font_manager_preview_pane_set_font`, `on_page_switch`) | Chamada a `font_manager_font_properties_page_cancel_rename()` ao selecionar outra fonte ou trocar de página no `GtkNotebook` | Baixo. Garante consistência visual e evita estado pendente entre seleções. |
| **Utilitários GTK** | `lib/gtk/font-manager-gtk-utils.h` | Reaproveitamento das macros e funções existentes (`FONT_MANAGER_DEFAULT_MARGIN`, `FONT_MANAGER_STYLE_CLASS_VIEW`, `font_manager_widget_set_margin()`) | Nenhum. Nenhuma alteração necessária no arquivo. |
| **Persistência e Fontconfig (futuro)** | `lib/fontconfig/font-manager-alias.c`, `lib/fontconfig/font-manager-fontconfig.c`, `lib/common/font-manager-database.c` | Não alterados no MVP; serão os pontos de integração para persistir o novo nome via `<alias>` ou atualização de metadados | Nenhum no MVP (Médio/Alto em evolução futura). |
| **Sistema de Build** | `lib/gtk/meson.build` | Nenhuma alteração necessária, pois apenas arquivos `.c` e `.h` já listados são modificados | Nenhum. |

### Riscos

- **Perda da alteração ao recarregar a fonte ou reiniciar o aplicativo:** Como o MVP atualiza o `JsonObject` da sessão sem persistir em arquivo/banco, trocar de fonte ou fechar o Font Manager restaura o nome original lido do Fontconfig/SQLite. Esse comportamento está explicitamente delimitado no escopo do MVP e deve ser documentado no Pull Request.
- **Vazamento de estado de edição entre fontes:** Se o usuário abrir o campo de renomeação em uma fonte e clicar em outra fonte na lista antes de confirmar, o campo poderia permanecer aberto com texto antigo. O risco é mitigado chamando `set_rename_mode(self, FALSE)` dentro de `reset()` e `font_manager_font_properties_page_cancel_rename()` em `font_manager_preview_pane_set_font()`.
- **Limitação de ambiente para compilação nativa:** Como o desenvolvimento da intervenção ocorre em ambiente Windows e o Font Manager depende de bibliotecas nativas Linux (GTK4, Pango, Fontconfig, SQLite), a validação do MVP concentra-se na revisão estática do código, casos de teste lógicos e revisão cruzada no Pull Request.