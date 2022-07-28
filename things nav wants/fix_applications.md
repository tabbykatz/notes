# Fix for newly discovered failed migration on Applications

## Background

Upon creating a new admin page, it was dicovered that while the thouroughly tested tool worked locally it failed in production. The cause was a call to ApplicationRepository to get `user_id`. This failed because `user_id` was not a column on Applicaiton in production, though it is in developement. With the help of Anthony Ruozzi, I learned that a migration to add `user_id` from at least a year ago had failed in production without notice but appeared successful in development. The reason I was the first to discover this problem is my unconventional query, where the preferred query would be on UserApplication or PreApplication. These child tables of Application both have the requisite columns, and this is why a fix is not straightforward.

Upon discussion with Anthony, Sandeep, and Nav, it was decided that it is preferable to not have the `user_id` column and that instead of fixing the migration we'd remove the column from development.

## Solution

I've been asked to drop the column if it exists without dropping the child tables' `user_id` columns. This migration should only run in dev if possible.

## First Attempt

```ruby
class RemoveUserIdFromApplications < ActiveRecord::Migration[6.0]
  def change
    remove_column :applications, :user_id, :integer
  end
end
```

### Before running the above migration:

```ruby
[1] pry(main)> Application.column_names.include? 'user_id'
=> true
[2] pry(main)> UserApplication.column_names.include? 'user_id'
=> true
[3] pry(main)> PreApplication.column_names.include? 'user_id'
=> true
[4] pry(main)>
```

### Warning

```ruby
=== Dangerous operation detected #strong_migrations ===

Active Record caches attributes which causes problems
when removing columns. Be sure to ignore the column:

class Application < ApplicationRecord
  self.ignored_columns = ["user_id"]
end

Deploy the code, then wrap this step in a safety_assured { ... } block.

class RemoveUserIdFromApplications < ActiveRecord::Migration[6.0]
  def change
    safety_assured { remove_column :applications, :user_id, :integer }
  end
end
```

```ruby
class RemoveUserIdFromApplications < ActiveRecord::Migration[6.0]
  def change
    safety_assured { remove_column :applications, :user_id, :integer }
  end
end

```

### After running this migration

```ruby
[5] pry(main)> Application.column_names.include? 'user_id'
=> false
[6] pry(main)> UserApplication.column_names.include? 'user_id'
=> false
[7] pry(main)> PreApplication.column_names.include? 'user_id'
=> false
```

### Result: Failure

Dropping the column on the parent removed it from the children as well.

## Another migration

This migration is only different in that it checks to see if the column exists, so I do not expect it to fix the problem of the previous attempt.

```ruby
if ActiveRecord::Base.connection.column_exists?(:applications , :user_id)
  change_table(:applications) do |t|
    t.remove :user_id
end
```

## Findings

I have not been able to drop this column without altering the child tables. According to [postgreSQL documentation](https://www.postgresql.org/docs/current/ddl-inherit.html), this is expected behaviour and cannot be prevented if the tables inherit from one another.

> `ALTER TABLE` will propagate any changes in column data definitions and check constraints down the inheritance hierarchy.
