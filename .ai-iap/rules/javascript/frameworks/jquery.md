# jQuery

> **Scope**: Apply these rules when working with jQuery for DOM manipulation and AJAX. Consider modern alternatives (vanilla JS, Alpine.js) for new projects.

## Overview

jQuery simplifies DOM manipulation and AJAX. Use for legacy projects only - consider vanilla JS or Alpine.js for new projects.

## Best Practices

**MUST**:
- Cache selectors (store in variables)
- Use event delegation for dynamic content
- Chain methods when possible

**SHOULD**:
- Namespace events for easy cleanup
- Use $.ajax() promises
- Consider vanilla JS for simple tasks

**AVOID**:
- jQuery for new projects (use modern alternatives)
- Repeated DOM queries
- $.each for arrays (use native forEach)

## 1. When to Use jQuery
- **Legacy Projects**: Maintaining existing jQuery codebases.
- **Quick Prototypes**: Rapid DOM manipulation.
- **Plugin Ecosystem**: Leveraging jQuery plugins.
- **Avoid For**: New SPAs (use React/Vue), modern projects (use vanilla JS).

## 2. Key Patterns

| Pattern | Example |
|---------|---------|
| **Cache Selectors** | `const $list = $('#list'); $list.append(item)` |
| **Chaining** | `$('#list').append(item).addClass('loaded').show()` |
| **Event Delegation** | `$(document).on('click', '.delete-btn', fn)` |
| **Namespaced Events** | `$el.on('click.plugin', fn); $el.off('click.plugin')` |
| **AJAX** | `$.ajax({ url, method, data }).done(fn).fail(fn)` |
| **DOM Creation** | `$('<div>', { class: 'card', html: content })` |
| **Data Attributes** | `$el.data('userId', 123); const id = $el.data('userId')` |

## 3. Plugin Pattern

```javascript
(function($) {
  $.fn.tooltip = function(options) {
    return this.each(function() {
      $(this).on('mouseenter', showTooltip).on('mouseleave', hideTooltip);
    });
  };
})(jQuery);
```

## 4. Modern Alternative

```javascript
// Vanilla JS (preferred for new projects)
document.querySelectorAll('.item').forEach(el => el.classList.add('active'));

// vs jQuery
$('.item').addClass('active');
```

