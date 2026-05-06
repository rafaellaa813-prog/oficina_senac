# Documentação de Especificação - Oficina Senac

## Visão Geral
Projeto: site institucional para oficina automotiva (Oficina Senac).
Objetivo: site responsivo com seções informativas, portfólio, contatos e navegação modular.

---

## Inventário de arquivos analisados
- `index.html` - página principal com seções: hero, serviços, marcas, sobre, projetos, preços, equipe, contato.
- `assets/css/microframework.css` - biblioteca de utilitários e componentes (BEM-like) com tokens, variáveis de tema, utilitários de layout, componentes (navbar, cards, forms, modal, botões).
- `assets/css/style.css` - arquivo com tokens CSS customizados: paleta, tipografia e algumas classes de botão.
- `assets/js/navbar.js` - gera as navbars (principal e secundária) dinamicamente, marca link ativo.
- `assets/js/footer.js` - tenta injetar HTML do rodapé dinamicamente (contém bugs — variáveis não definidas e uso de template mal formado).
## Análise crítica (problemas encontrados)

- JavaScript:
  - `footer.js`: referência a `footerContainer` indefinida (deve usar `footer`). Isso causa erro JS e impede renderização do rodapé.
  - Injeção de grandes blocos HTML via `innerHTML` dificulta manutenção, teste unitário e reuso.
  - Repetição de SVGs inline no `footer.js` aumenta o tamanho do template — mover para componentes ou sprites recomendado.
  - Ausência de tratamento de erros/feedback em interações (ex.: formulários, fetches).

- CSS:
  - Duplicação de tokens entre `assets/css/style.css` e `assets/css/microframework.css` leva a inconsistência e dificuldades de manutenção. Recomendo centralizar tokens em `microframework.css` e importar/estender quando necessário.
  - Regras com valores fixos (`width`, `height` em IMG) reduzem responsividade; prefira `max-width: 100%` e `height: auto` para imagens responsivas.
  - Cursor URL aponta para `../asset/imagem/cursor.svg` (possível erro de digitação — pasta `assets`), o que pode quebrar o cursor personalizado.
  - Contraste e foco: não há garantias de contraste entre texto e fundo em todos os componentes; foco de elementos (`:focus`) usa `opacity:.01` em alguns seletores, isso é problemático para usabilidade.

- HTML:
  - Dependência da injeção JS para elementos essenciais (nav, footer) impede navegação quando JS está desativado; considerar marcação server-rendered ou inline como fallback.
  - Estrutura de algumas seções está incompleta (ex.: `id="mecanica-geral"` deve evitar espaços) — ids não devem conter espaços.
  - Marcação semântica pode ser melhorada: usar `<address>`, `<dl>` para FAQs, `<figure>`/`<figcaption>` para projetos, etc.

---

## Recomendações e Sugestões de Melhoria (priorizadas)

1. Correções críticas (faça primeiro):
   - Corrigir `assets/js/footer.js` (usar `footer` em vez de `footerContainer`) para restaurar o rodapé.
   - Corrigir caminhos estáticos (`cursor.svg`, imagens) e nomes de pastas inconsistentes.
   - Remover ou reduzir injeção massiva de HTML quando desnecessário; modularizar templates (pequenas funções que retornam elementos) ou usar `<template>`.

2. Arquitetura e manutenção:
   - Consolidar tokens CSS: mover variáveis do `style.css` para `microframework.css` e utilizar apenas um conjunto de tokens.
   - Padronizar nomenclatura: manter o prefixo `bem-` (ou migrar tudo para outro padrão) e evitar classes ad-hoc como `price`/`button` sem prefixo.

3. Acessibilidade e semântica:
   - Implementar foco visível consistente e evitar `opacity:.01` em estados de foco.
   - Melhorar landmarks e usar roles/aria onde necessário (ex.: nav role, landmarks no footer, aria-controls/aria-expanded para accordions).

4. Performance:
   - Otimizar imagens (WebP/AVIF), usar `loading="lazy"` em imagens fora do viewport.
   - Extrair SVGs repetidos para sprites ou usar symbol/defs para evitar duplicação.

5. Experiência do desenvolvedor:
   - Adicionar linting (ESLint, stylelint), e um pequeno script npm com ferramentas de build (opcional).

---

## Tokens e Diretrizes Visuais (extraídas dos arquivos)
- Paleta principal (do `microframework.css`): `--bem-primary`, `--bem-terciary` (#f59e0b), `--bem-dark` (#1f2937), `--bem-light` (#f3f4f6).
- Paleta no `style.css`: `--color-accent: #FFB800` (dourado), `--color-primary-dark: #1A1C1E`.
- Tipografia sugerida: variável `--bem-font-base` e em `style.css` `--font-main: 'Inter'`, `--font-heading: 'Montserrat'` — alinhar fontes e fallback.

Recomendação: adotar `--color-accent` = `--bem-terciary` para consistência e usar `--bem-font-base` como fonte padrão.

---

## Plano de Implementação: Seção de Perguntas Frequentes (FAQ)

Objetivo: adicionar uma seção FAQ acessível, leve, com comportamento accordion para telas mobile e desktop.

Requisitos:
- A seção deve ser navegável por teclado.
- Cada item do accordion deve usar `button` com `aria-expanded` e `aria-controls`.
- Colapso/expansão animada (altura) sem depender de bibliotecas externas.

Markup sugerido (exemplo):

```html
<section id="faq" class="bem-container bem-pb-xl">
  <h2 class="bem-titulo-h2">Perguntas Frequentes</h2>
  <div class="faq-list">
    <div class="faq-item">
      <button class="faq-toggle bem-btn bem-btn--outline" aria-expanded="false" aria-controls="faq-1" id="faq-btn-1">
        Como solicito um orçamento?
        <span class="faq-icon">+</span>
      </button>
      <div id="faq-1" class="faq-panel" role="region" aria-labelledby="faq-btn-1" hidden>
        <p>Para solicitar um orçamento, acesse a página de contato ou clique em "Orçamento".</p>
      </div>
    </div>
    <!-- repetir itens -->
  </div>
</section>
```

CSS mínimo (padrões `bem-`):

```css
.faq-toggle { display:flex; justify-content:space-between; width:100%; }
.faq-panel { transition: max-height .28s ease; overflow:hidden; }
.faq-panel[hidden] { display:none; }
```

JS (comportamento acessível):

```javascript
document.querySelectorAll('.faq-toggle').forEach(btn => {
  btn.addEventListener('click', () => {
    const expanded = btn.getAttribute('aria-expanded') === 'true';
    const panel = document.getElementById(btn.getAttribute('aria-controls'));
    btn.setAttribute('aria-expanded', !expanded);
    if (expanded) {
      panel.hidden = true;
    } else {
      panel.hidden = false;
      panel.focus();
    }
  });
});
```

Notas:
- Use `hidden` para esconder conteúdos e `role="region" aria-labelledby` para ajudar leitores de tela.
- Para animação de altura, calcular `max-height` via JS ou usar `height` com `requestAnimationFrame`.

---

## Plano de Implementação: Formulário Moderno na página externa `contato.html`

Objetivo: criar um formulário responsivo, acessível e visualmente alinhado às cores/tipografia do `index.html`.

Campos recomendados:
- Nome completo (`name`) — obrigatório
- Email (`email`) — obrigatório, type=email
- Telefone (`tel`) — opcional, pattern
- Tipo de serviço (`select`) — opcional
- Mensagem (`textarea`) — obrigatório
- Checkbox de consentimento (`privacy`) — obrigatório
- Botão de envio com estado (`Enviar`, `Enviando...`, `Enviado`).

Exemplo de marcação:

```html
<form id="contact-form" class="bem-form bem-container--narrow" novalidate>
  <div class="bem-form__group">
    <label class="bem-form__label bem-form__label--required" for="nome">Nome completo</label>
    <input id="nome" name="nome" class="bem-form__input" required>
  </div>
  <div class="bem-form__group">
    <label class="bem-form__label bem-form__label--required" for="email">Email</label>
    <input id="email" name="email" type="email" class="bem-form__input" required>
  </div>
  <div class="bem-form__group">
    <label class="bem-form__label" for="telefone">Telefone</label>
    <input id="telefone" name="telefone" type="tel" class="bem-form__input">
  </div>
  <div class="bem-form__group">
    <label class="bem-form__label" for="mensagem">Mensagem</label>
    <textarea id="mensagem" name="mensagem" class="bem-form__textarea" required></textarea>
  </div>
  <div class="bem-form__group">
    <label class="bem-form__checkbox"><input type="checkbox" id="privacy" required> Concordo com o uso dos dados</label>
  </div>
  <button type="submit" class="bem-btn bem-btn--primary">Enviar</button>
  <div id="form-feedback" class="bem-form__help" aria-live="polite"></div>
</form>
```

Estilo (sugestão):

```css
.bem-form__input, .bem-form__textarea { border-radius: var(--bem-radius-md); border:1px solid var(--bem-border); }
.bem-btn--primary { background-color: var(--bem-terciary); color:#fff; }
.bem-form__help { margin-top: .5rem }
```

JS de validação e submissão (exemplo com fetch):

```javascript
const form = document.getElementById('contact-form');
const feedback = document.getElementById('form-feedback');
form.addEventListener('submit', async (e) => {
  e.preventDefault();
  if (!form.checkValidity()) {
    feedback.textContent = 'Por favor, complete os campos obrigatórios.';
    return;
  }
  const data = Object.fromEntries(new FormData(form));
  try {
    const btn = form.querySelector('button[type="submit"]');
    btn.disabled = true; btn.textContent = 'Enviando...';
    const res = await fetch('https://formspree.io/f/yourformid', {
      method: 'POST', headers: { 'Accept': 'application/json' }, body: JSON.stringify(data)
    });
    if (res.ok) {
      feedback.textContent = 'Mensagem enviada com sucesso.';
      form.reset();
    } else {
      const json = await res.json();
      feedback.textContent = json.error || 'Ocorreu um erro ao enviar.';
    }
    btn.disabled = false; btn.textContent = 'Enviar';
  } catch (err) {
    feedback.textContent = 'Erro de rede. Tente novamente.';
  }
});
```

Integração:
- Backend próprio: criar endpoint `/api/contact` que envie email via SMTP ou serviço (SendGrid, SES).
- Sem backend: usar Formspree, Getform, Netlify Forms ou Google Forms.

---

## Correções rápidas sugeridas (patches mínimos)
- Corrigir `footer.js`: trocar `footerContainer.innerHTML =` por `footer.innerHTML =`.
- Validar `id` sem espaços (`id="mecanica-geral"` ao invés de `mecanica geral`).
- Ajustar `cursor` path em `microframework.css` para `url("../assets/imagens/cursor.svg")` ou remover para confiabilidade.

---

## Plano de trabalho (tasks) — implementação prática
- 1. Corrigir `footer.js` e validar console (10–20 min).
- 2. Ajustar tokens CSS e documentar mudanças (30–60 min).
- 3. Implementar FAQ (HTML/CSS/JS) na `index.html` (ou arquivo partial) (30–60 min).
- 4. Implementar formulário moderno em `contato.html` (HTML/CSS/JS) e integrar com Formspree ou endpoint (45–90 min).
- 5. Testes de acessibilidade e responsividade (30–60 min).

---

## Conclusão
O projeto tem uma boa base: um microframework CSS consistente e uso inteligente de SVGs e componentes injetados. Com pequenas correções (rodapé JS, tokens duplicados, melhorias de acessibilidade) é possível tornar o código mais robusto e acelerar futuras funcionalidades (FAQ, formulários, integrações).

Se desejar, posso aplicar automaticamente as correções críticas (rodapé, id com espaços, criar a seção FAQ e o formulário de `contato.html`) — diga quais itens quer que eu implemente primeiro.

---

Arquivo gerado automaticamente: `DOCUMENTACAO-PROJETO.md` — use este documento como especificação para novas implementações por IA ou desenvolvedores.
