# Investigação: Conflito de Posicionamento do Navbar

## Status: INVESTIGAÇÃO COMPLETA
Data: 04 de maio de 2026
Agente: GitHub Copilot

---

## 📋 Resumo Executivo

A implementação de `navbar.js` quebrou o posicionamento dos navbars porque **há duas definições CSS conflitantes** para `.bem-navbar` e uma definição parcialmente incorreta para `.bem-navbar--secundaria` no arquivo `microframework.css`.

**Conflitos Identificados:** 3 maiores
**Gravidade:** ALTA - Afeta navegação principal da aplicação
**Pré-requisito para Correção:** Consolidação de CSS duplicado

---

## 🔴 Conflitos Identificados

### CONFLITO 1: `.bem-navbar` Definido Duas Vezes (Duplicação)

#### Localização 1 (Linhas 131-143)
```css
.bem-navbar{
  display: flex;
  align-items: center;
  justify-content: space-around;
  padding: var(--bem-spacing-md) var(--bem-spacing-lg);
  box-shadow: var(--bem-shadow-sm);
  transition: background .25s;
  position: sticky;        /* ❌ INCORRETO: deve ser fixed */
  top: 0;                  /* ✓ Correto */
  background-color: var(--bem-dark);
  color: var(--bem-dark);
  z-index: 100;
  width: 100%;
}
```

#### Localização 2 (Linha 370-379)
```css
.bem-navbar{
  display:flex;
  align-items:center;
  justify-content:space-between;
  padding:var(--bem-spacing-md) var(--bem-spacing-lg);
  background-color:var(--bem-surface);
  box-shadow:var(--bem-shadow-sm);
  border-bottom:1px solid var(--bem-border);
  transition:background .25s;
  position:sticky; top:0; z-index:100;  /* ❌ INCORRETO: deve ser fixed */
}
```

**Problema:** A segunda definição sobrescreve a primeira (cascata CSS). Ambas usam `position: sticky` quando o requisito é `position: fixed`.

**Impacto:** O navbar principal não fica fixo no topo da página durante scroll. Em vez disso, sai do viewport.

---

### CONFLITO 2: `.bem-navbar` vs `.bem-navbar--secundaria` - Hierarquia Z-Index Invertida

#### `.bem-navbar` (Navbar Principal)
- **Linha 142 (primeira definição):** `z-index: 100;`
- **Linha 379 (segunda definição):** `z-index: 100;`
- **Comportamento atual:** Sticky (sai da tela durante scroll)

#### `.bem-navbar--secundaria` (Navbar Secundária)
- **Linha 152:** `position: sticky; top: 70px; z-index: 10;`
- **Comportamento esperado:** Sticky a 70px do topo (abaixo do navbar principal)
- **z-index:** Corretamente menor (10) que navbar principal (100)

**Problema:** Se navbar principal fosse fixed corretamente, secundária deveria ter `position: sticky; top: 70px`. Está correto, MAS o navbar principal está quebrado, então toda a hierarquia falha.

**Impacto:** Layout não mantém a hierarquia visual esperada.

---

### CONFLITO 3: `justify-content` Inconsistente Entre Definições

#### Primeira definição (Linha 131)
```css
justify-content: space-around;
```

#### Segunda definição (Linha 370)
```css
justify-content: space-between;
```

**Problema:** A segunda definição sobrescreve a primeira. Não está claro qual é a intenção de design.

**Impacto:** Espaçamento do conteúdo do navbar principal pode estar incorreto em relação ao design original.

---

## ✅ Requisitos de Design (Confirmados)

Com base na estrutura do site e nas tags HTML:

```html
<div id="navbar-principal"></div>      <!-- Deve ser position: fixed; top: 0 -->
<div id="navbar-secundaria"></div>     <!-- Deve ser position: sticky; top: 70px -->
```

### Navbar Principal (`.bem-navbar`)
- **Requisito:** `position: fixed; top: 0;`
- **Motivo:** Fica fixo no topo durante toda navegação
- **z-index:** 100 (apropriado, acima de conteúdo)
- **Altura esperada:** ~70px (para calcular offset do secundária)

### Navbar Secundária (`.bem-navbar--secundaria`)
- **Requisito:** `position: sticky; top: 70px;`
- **Motivo:** Fica fixo 70px abaixo do topo (abaixo do principal), sai quando section passa por ele
- **z-index:** 10 (correto, abaixo do principal)
- **Width:** Deve ser 100% para ocupar toda largura

---

## 🔍 Causa-Raiz: Por Que `navbar.js` Expôs Este Problema?

1. **Antes (HTML Estático):**
   - Navbars eram renderizados via HTML estático em cada página
   - CSS poderia estar sendo sobrescrito por inline styles ou ordem de carregamento
   - O problema existia mas não era óbvio

2. **Agora (Injeção via JavaScript):**
   - `navbar.js` injeta os elementos dinamicamente com `innerHTML`
   - A injeção ocorre no `DOMContentLoaded`
   - CSS já foi processado pelo navegador quando HTML é injetado
   - **Cascata CSS agora segue rigorosamente a ordem de precedência**
   - Como há duas definições de `.bem-navbar`, a segunda (linha 370) vence
   - A segunda está errada → navbar quebrado

**Conclusão:** O problema CSS sempre existiu, mas a renderização estática mascarava. A injeção dinâmica revelou o erro.

---

## 📊 Plano de Mudança (Antes da Implementação)

### Mudança 1: Consolidar `.bem-navbar` em Uma Única Definição
**Arquivo:** `assets/css/microframework.css`
**Ação:** Remover duplicação, manter uma definição com:
- `position: fixed;` (não sticky)
- `top: 0;`
- `z-index: 100;`
- `justify-content: space-between;` (verificar qual é correto com designer)
- `width: 100%;`
- Manter todos outros estilos: flex, padding, background, shadow, border-bottom

### Mudança 2: Confirmar `.bem-navbar--secundaria`
**Arquivo:** `assets/css/microframework.css`
**Status:** Já correto em linha 152
**Verificação:** Deixar como está se:
- `position: sticky; top: 70px;` ✓
- `z-index: 10;` ✓
- `width: 100%;` (verificar se presente)

### Mudança 3: Verificar Conflitos em Arquivo `style.css`
**Arquivo:** `assets/css/style.css`
**Ação:** Buscar se há override adicional de `.bem-navbar` que sobrescreva positioning

---

## 🧪 Plano de Teste (antes de commit)

1. **Abrir na URL:** `http://127.0.0.1:5500`
2. **Teste 1 - Navbar Principal:**
   - ✓ Navbar principal fica fixo no topo durante scroll
   - ✓ Não sai do viewport
   - ✓ Z-index não permite conteúdo sobrepor
3. **Teste 2 - Navbar Secundária:**
   - ✓ Inicia 70px abaixo do topo
   - ✓ Permanece sticky ao fazer scroll em seções
   - ✓ Sai de viewport quando seção passa por ele
4. **Teste 3 - Layout:**
   - ✓ Conteúdo começa abaixo dos navbars
   - ✓ Nenhum conteúdo fica sob os navbars
   - ✓ Espaçamento horizontal é consistente

---

## 📝 Próximos Passos

1. ✅ Investigação completa - **PRONTO**
2. ⏳ Aguardar aprovação do plano de mudança
3. ⏳ Implementar Mudança 1 (consolidar `.bem-navbar`)
4. ⏳ Verificar Mudança 2 (confirmar secundária)
5. ⏳ Procurar Mudança 3 (verificar `style.css`)
6. ⏳ Testar em localhost:5500
7. ⏳ Validar com usuário

---

## 📚 Referências

- **navbar.js:** Renderiza `<nav class="bem-navbar">` e `<nav class="bem-navbar--secundaria">`
- **index.html:** Injeta em `<div id="navbar-principal">` e `<div id="navbar-secundaria">`
- **Duplicação identificada:** Linhas 131-143 vs 370-379 do microframework.css
