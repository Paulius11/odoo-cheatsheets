# Odoo 17 Mail Composer Cheatsheet

## Basic Usage

### Creating a Simple Mail Composer
```python
composer = self.env['mail.compose.message'].create({
    'subject': 'Subject line',
    'body': '<p>Message content</p>',
    'partner_ids': [(4, partner.id)],  # Add recipients
})
```

### Composition Modes
```python
# Comment Mode (Post on record's chatter)
composer = self.env['mail.compose.message'].with_context(
    default_model='sale.order',
    default_res_id=sale_order.id,
    default_composition_mode='comment'
).create({...})

# Mass Mail Mode (Send emails to multiple recipients)
composer = self.env['mail.compose.message'].with_context(
    default_model='res.partner',
    default_res_ids=[1, 2, 3],
    default_composition_mode='mass_mail'
).create({...})
```

## Message Types and Subtypes

### Comment Message (Visible to all followers)
```python
composer = self.env['mail.compose.message'].with_context(
    default_subtype_xmlid='mail.mt_comment',
    default_model='sale.order',
    default_res_id=order.id
).create({
    'body': 'Public comment visible to all followers',
    'partner_ids': [(4, partner.id)],
})
```

### Internal Note (Internal users only)
```python
composer = self.env['mail.compose.message'].with_context(
    default_subtype_xmlid='mail.mt_note',
    default_model='crm.lead',
    default_res_id=lead.id
).create({
    'body': 'Internal note for staff only',
})
```

## Template Usage

### Using Email Template
```python
composer = self.env['mail.compose.message'].with_context(
    default_model='sale.order',
    default_res_id=order.id,
    default_template_id=template.id
).create({})

# Force template language
composer = composer.with_context(lang='fr_FR')
```

### Creating Template from Composer
```python
composer.write({
    'subject': 'Template Subject',
    'body': 'Template Content'
})
composer.create_mail_template()
```

## Advanced Features

### Attachments
```python
# Add attachments
composer.write({
    'attachment_ids': [
        (0, 0, {
            'name': 'document.pdf',
            'datas': base64_encoded_data,
            'type': 'binary'
        })
    ]
})

# Remove attachment
composer.write({
    'attachment_ids': [(3, attachment.id)]
})
```

### Scheduling
```python
composer.write({
    'scheduled_date': '2024-12-25 10:00:00'
})
```

### Reply Management
```python
composer.write({
    'reply_to': 'custom@example.com',
    'reply_to_force_new': True,  # Start new thread
    'reply_to_mode': 'new'  # 'new' or 'update'
})
```

## Mass Mailing Features

### Managing Recipients
```python
composer.write({
    'use_exclusion_list': True,  # Check blacklist
    'auto_delete': True,  # Delete sent emails
    'auto_delete_keep_log': True,  # Keep message copy
})
```

### Batch Processing
```python
# Using domain for recipients
composer = self.env['mail.compose.message'].with_context(
    default_model='res.partner',
    default_composition_mode='mass_mail'
).create({
    'res_domain': "[('customer_rank', '>', 0)]"
})
```

## Common Context Keys

```python
context = {
    'default_model': 'model.name',  # Target model
    'default_res_id': record.id,  # Single record ID
    'default_res_ids': [1, 2, 3],  # Multiple record IDs
    'default_composition_mode': 'comment',  # or 'mass_mail'
    'default_template_id': template.id,  # Email template
    'default_subtype_xmlid': 'mail.mt_comment',  # Message subtype
    'default_partner_ids': [(4, partner.id)],  # Recipients
    'mail_post_autofollow': True,  # Auto-follow on post
    'mail_create_nosubscribe': True,  # Don't subscribe author
}
```

## Error Handling and Validation
```python
try:
    composer.action_send_mail()
except UserError as e:
    # Handle user errors (e.g., missing recipients)
    _logger.error("Mail composer error: %s", e)
except ValidationError as e:
    # Handle validation errors
    _logger.error("Validation error: %s", e)
```

## Useful Methods

### Send Mail
```python
# Simple send
composer.action_send_mail()

# Send with auto-commit
composer._action_send_mail(auto_commit=True)
```

### Template Operations
```python
# Preview template
values = composer._generate_template_for_composer(
    res_ids,
    ['subject', 'body_html', 'email_to']
)

# Update from template
composer._set_value_from_template('subject')
composer._set_value_from_template('body_html', 'body')
```

## Best Practices

1. Always handle exceptions when sending emails
2. Use appropriate subtypes for different message purposes
3. Consider using templates for consistent communication
4. Test mass mailings with small batches first
5. Handle attachments properly to avoid memory issues
6. Use appropriate composition mode for the use case
7. Consider email scheduling for optimal delivery times
8. Manage reply-to settings based on thread requirements
9. Use context keys for proper initialization
10. Validate recipients before sending mass emails

