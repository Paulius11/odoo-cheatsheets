# Odoo Many2Many Commands Cheat Sheet

## Basic Command Structure

### Old Way (Before Odoo 14)
```python
# Basic structure: (command_code, id, values)
[(code, id, values)]
```

### New Way (Odoo 14+)
```python
# Using Command class
Command.operation(parameters)
```

## Command Mapping

| Operation        | Old Way                  | New Way (Command)          | Description                           |
|-----------------|--------------------------|---------------------------|---------------------------------------|
| Add Link        | `[(4, id, 0)]`          | `Command.link(id)`        | Links existing record                 |
| Remove Link     | `[(3, id, 0)]`          | `Command.unlink(id)`      | Removes link without deleting record  |
| Create & Link   | `[(0, 0, values)]`      | `Command.create(values)`  | Creates new record and links it       |
| Update Linked   | `[(1, id, values)]`     | `Command.update(id, values)` | Updates existing linked record     |
| Remove & Delete | `[(2, id, 0)]`          | `Command.delete(id)`      | Removes link and deletes record       |
| Replace All     | `[(6, 0, [ids])]`       | `Command.set([ids])`      | Replaces all links with new ones      |
| Clear All       | `[(5, 0, 0)]`           | `Command.clear()`         | Removes all links                     |

## XML Examples

### Old Way
```xml
<!-- Adding users to group -->
<field name="users" eval="[(4, ref('base.user_admin'), 0)]"/>

<!-- Replace all users -->
<field name="users" eval="[(6, 0, [ref('base.user_admin'), ref('base.user_root')])]"/>

<!-- Create and link new record -->
<field name="user_ids" eval="[(0, 0, {'name': 'New User', 'login': 'new@example.com'})]"/>

<!-- Remove link -->
<field name="users" eval="[(3, ref('base.user_demo'), 0)]"/>
```

### New Way
```xml
<!-- Adding users to group -->
<field name="users" eval="[Command.link(ref('base.user_admin'))]"/>

<!-- Replace all users -->
<field name="users" eval="[Command.set([ref('base.user_admin'), ref('base.user_root')])]"/>

<!-- Create and link new record -->
<field name="user_ids" eval="[Command.create({'name': 'New User', 'login': 'new@example.com'})]"/>

<!-- Remove link -->
<field name="users" eval="[Command.unlink(ref('base.user_demo'))]"/>
```

## Python Examples

### Old Way
```python
# Add a single user
group.write({'users': [(4, user_id, 0)]})

# Replace all users
group.write({'users': [(6, 0, [user_id1, user_id2])]})

# Create and link
group.write({'users': [(0, 0, {'name': 'New User'})]})

# Remove link
group.write({'users': [(3, user_id, 0)]})
```

### New Way
```python
from odoo.fields import Command

# Add a single user
group.write({'users': [Command.link(user_id)]})

# Replace all users
group.write({'users': [Command.set([user_id1, user_id2])]})

# Create and link
group.write({'users': [Command.create({'name': 'New User'})]})

# Remove link
group.write({'users': [Command.unlink(user_id)]})
```

## Common Combinations

### Old Way
```python
# Add multiple links
[(4, id1, 0), (4, id2, 0)]

# Create multiple records
[(0, 0, vals1), (0, 0, vals2)]

# Update and add
[(1, id1, vals), (4, id2, 0)]
```

### New Way
```python
# Add multiple links
[Command.link(id1), Command.link(id2)]

# Create multiple records
[Command.create(vals1), Command.create(vals2)]

# Update and add
[Command.update(id1, vals), Command.link(id2)]
```

## Best Practices

1. **Always use the new Command class** in new code for better readability
2. **Import Command properly**:
   ```python
   from odoo.fields import Command
   ```
3. **Use meaningful variable names** for IDs and values
4. **Group related operations** together
5. **Document complex combinations** of commands
6. **Handle errors** appropriately when records don't exist
7. **Consider performance** with large datasets

## Common Gotchas

1. Don't mix old and new syntax in the same operation
2. Remember that Command.set() replaces ALL existing links
3. Command.link() silently handles duplicates
4. Always use list brackets even for single commands
5. XML IDs must exist when using ref() function