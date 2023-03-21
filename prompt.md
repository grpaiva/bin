#### User:

In my laravel application I have this "Company" model that has a "credits" relation:

```
<?php
namespace Descola\Models;

use Illuminate\Database\Eloquent\Model;

class Company extends Model
{
    
    (...)
    
    public function credits()
    {
        return $this->hasMany(Credit::class, 'owner_id')
            ->where('credits.owner_type', Credit::OWNER_TYPE_COMPANY);
    }
    
    (...)
    
}
```

And this is my "Credit" class:

```
<?php
namespace Descola\Models;

use Illuminate\Database\Eloquent\Model;

class Credit extends Model
{
    public const STATUS_INACTIVE = 0;
    public const STATUS_ACTIVE = 1;

    public const OWNER_TYPE_USER = 1;
    public const OWNER_TYPE_COMPANY = 2;
    public const OWNER_TYPE_TEAM = 3;

    protected $fillable = [
        'product_id', 'owner_id', 'owner_type', 'user_id', 'company_id',
        'team_id', 'order_id', 'unit_price', 'quantity_buyed', 'quantity_used', 'status',
    ];

    (...)
    
}
```

I want to query all Companies, with pagination, adding each company's credits balance and ordering by this balance in descending order.


#### Assistant: 

You can achieve this by using Laravel's query builder and adding a subquery to calculate the credit balance for each company. Here's an example:

```
use Illuminate\Support\Facades\DB;

$companies = DB::table('companies')
    ->select('companies.*', DB::raw('(SELECT SUM(quantity_buyed - quantity_used) FROM credits WHERE owner_id = companies.id AND owner_type = ' . Credit::OWNER_TYPE_COMPANY . ') AS credit_balance'))
    ->orderByDesc('credit_balance')
    ->paginate(10);
```

In this query, we first select all columns from the `companies` table. Then we add a subquery using `DB::raw()` to calculate the credit balance for each company. The subquery uses the `SUM()` function to calculate the difference between the `quantity_buyed` and `quantity_used` columns for all credits belonging to the current company. We also filter the credits by `owner_type` and `owner_id` to only include credits belonging to companies.

Finally, we order the results by `credit_balance` in descending order and paginate the results using `paginate(10)`.

Note that this query assumes that the `credits` table has a foreign key `owner_id` referencing the `id` column of the `companies` table. If your schema is different, you may need to adjust the query accordingly.
