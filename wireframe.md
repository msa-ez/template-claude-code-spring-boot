forEach: Aggregate
path: {{#boundedContext}}{{name}}{{/boundedContext}}/{{name}}
fileName: {{name}}Wireframe.md
---

In the process of creating components related to {{name}} Service, UI for buttons, modals, dashboards, and search bars according to Command and ReadModel have been represented in HTML.
The style and text of the HTML below should be used as-is, but only the code format should be changed to React to create functional components.

{{#attached 'Command' this}}
{{#attached 'UI' this}}
Name: {{../name}}
API Endpoint: /{{../aggregate.namePlural}}/{{../controllerInfo.apiPath}}
```
{{#changeData runTimeTemplateHtml}}{{/changeData}}
```

{{/attached}}
{{/attached}}
{{#attached 'View'this}}
{{#attached 'UI' this}}
Name: {{../name}}
```
{{#changeData runTimeTemplateHtml}}{{/changeData}}
```
{{/attached}}
{{/attached}}

<function>
window.$HandleBars.registerHelper('changeData', function (desc) {
    // Decode HTML entities into actual HTML characters
    if (typeof desc === 'string') {
        let decodedHtml = desc
            .replace(/&lt;/g, '<')
            .replace(/&gt;/g, '>')
            .replace(/&quot;/g, '"')
            .replace(/&#x3D;/g, '=')
            .replace(/&amp;/g, '&')
            .replace(/&#39;/g, "'")
            .replace(/&nbsp;/g, ' ');
        
        // HTML formatting function for readable code output
        function formatHtml(html) {
            let formatted = '';
            let indent = 0;
            const indentStr = '  '; // 2-space indentation
            
            // Block-level elements
            const blockTags = ['div', 'section', 'article', 'header', 'footer', 'main', 'nav', 'aside', 'form', 'fieldset', 'table', 'thead', 'tbody', 'tr', 'ul', 'ol', 'li', 'dl', 'dt', 'dd', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6', 'p', 'blockquote', 'pre', 'address'];
            
            // Self-closing elements
            const selfClosingTags = ['img', 'br', 'hr', 'input', 'meta', 'link', 'area', 'base', 'col', 'embed', 'source', 'track', 'wbr'];
            
            // Split HTML into tokens (tags and text content)
            const tokens = html.match(/<\/?[^>]+>|[^<]+/g) || [];
            
            for (let i = 0; i < tokens.length; i++) {
                const token = tokens[i].trim();
                if (!token) continue;
                
                if (token.startsWith('</')) {
                    // Closing tag
                    indent--;
                    formatted += indentStr.repeat(Math.max(0, indent)) + token + '\n';
                } else if (token.startsWith('<')) {
                    // Opening tag or self-closing tag
                    const tagMatch = token.match(/<(\w+)/);
                    const tagName = tagMatch ? tagMatch[1].toLowerCase() : '';
                    
                    formatted += indentStr.repeat(indent) + token + '\n';
                    
                    // Increase indentation for non-self-closing block tags
                    if (!selfClosingTags.includes(tagName) && !token.endsWith('/>')) {
                        indent++;
                    }
                } else {
                    // Text content
                    if (token.length > 0) {
                        formatted += indentStr.repeat(indent) + token + '\n';
                    }
                }
            }
            
            return formatted.trim();
        }
        
        // Apply HTML formatting for readable output
        return formatHtml(decodedHtml);
    }
    return desc;
})
</function>