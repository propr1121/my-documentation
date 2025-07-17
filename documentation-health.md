---
title: Documentation Health
layout: default
nav_order: 14
last_modified_date: 2025-07-17
---

{% include freshness_indicator.html %}

# 📊 Documentation Health

This dashboard provides real-time insights into the health, freshness, and completeness of the Stream Systems documentation suite.

{% include content_gaps_audit.html %}

## 📈 Documentation Quality Metrics

### Freshness Standards
- 🟢 **Fresh (0-30 days)**: Recently updated, current information
- 🟡 **Aging (31-90 days)**: Should be reviewed for accuracy
- 🔴 **Needs Update (90+ days)**: Requires immediate review and updating

### Coverage Standards
- **80%+ Coverage**: Excellent documentation suite
- **60-79% Coverage**: Good but missing some key areas
- **Below 60%**: Significant gaps requiring attention

## 📋 Page-by-Page Status

<div style="background: #f8f9fa; border: 1px solid #e9ecef; border-radius: 8px; padding: 1.5rem; margin: 2rem 0;">

### Core Documentation Pages

{% assign core_pages = site.pages | where_exp: "page", "page.path contains '.md'" %}

<table style="width: 100%; border-collapse: collapse; margin-top: 1rem;">
  <thead>
    <tr style="background: #f3f4f6;">
      <th style="padding: 0.75rem; text-align: left; border: 1px solid #e5e7eb;">Page</th>
      <th style="padding: 0.75rem; text-align: center; border: 1px solid #e5e7eb;">Last Updated</th>
      <th style="padding: 0.75rem; text-align: center; border: 1px solid #e5e7eb;">Status</th>
      <th style="padding: 0.75rem; text-align: center; border: 1px solid #e5e7eb;">Days Old</th>
    </tr>
  </thead>
  <tbody>
    {% for page in core_pages %}
      {% if page.title and page.title != "" %}
        {% assign today = 'now' | date: '%s' %}
        {% assign page_date = page.last_modified_date | default: page.date | date: '%s' %}
        {% assign days_old = today | minus: page_date | divided_by: 86400 %}
        
        {% if days_old <= 30 %}
          {% assign status_color = '#22c55e' %}
          {% assign status_text = 'Fresh' %}
          {% assign status_icon = '🟢' %}
        {% elsif days_old <= 90 %}
          {% assign status_color = '#f59e0b' %}
          {% assign status_text = 'Aging' %}
          {% assign status_icon = '🟡' %}
        {% else %}
          {% assign status_color = '#ef4444' %}
          {% assign status_text = 'Needs Update' %}
          {% assign status_icon = '🔴' %}
        {% endif %}
        
        <tr style="{% cycle 'background: white;', 'background: #f9fafb;' %}">
          <td style="padding: 0.75rem; border: 1px solid #e5e7eb;">
            <a href="{{ page.url }}" style="color: #374151; text-decoration: none; font-weight: 500;">{{ page.title }}</a>
          </td>
          <td style="padding: 0.75rem; border: 1px solid #e5e7eb; text-align: center; font-size: 0.875rem;">
            {{ page.last_modified_date | default: page.date | date: "%b %d, %Y" }}
          </td>
          <td style="padding: 0.75rem; border: 1px solid #e5e7eb; text-align: center;">
            <span style="background: {{ status_color }}15; color: {{ status_color }}; padding: 0.25rem 0.5rem; border-radius: 4px; font-size: 0.75rem; font-weight: 500;">
              {{ status_icon }} {{ status_text }}
            </span>
          </td>
          <td style="padding: 0.75rem; border: 1px solid #e5e7eb; text-align: center; font-size: 0.875rem; color: #6b7280;">
            {{ days_old }}
          </td>
        </tr>
      {% endif %}
    {% endfor %}
  </tbody>
</table>

</div>

## 🎯 Recommendations

### Immediate Actions Required
1. **Update aging documents** (yellow status) within 2 weeks
2. **Prioritize red status pages** for immediate review
3. **Create missing critical documentation** identified in the gaps analysis

### Maintenance Schedule
- **Weekly**: Review fresh documents for accuracy
- **Monthly**: Update aging documents
- **Quarterly**: Comprehensive documentation audit
- **Annually**: Complete documentation strategy review

## 📝 Documentation Standards

### Quality Checklist
- [ ] Clear, actionable content
- [ ] Up-to-date screenshots and examples
- [ ] Proper cross-linking between related pages
- [ ] Contact information for subject matter experts
- [ ] Review dates and next review schedule

### Template Compliance
- [ ] Consistent formatting and styling
- [ ] Proper frontmatter with last_modified_date
- [ ] Freshness indicators on all pages
- [ ] Appropriate navigation order

---

*Dashboard last updated: {{ page.last_modified_date | date: "%B %d, %Y" }}*
*Next scheduled review: {{ page.last_modified_date | date: "%s" | plus: 2592000 | date: "%B %d, %Y" }}* 