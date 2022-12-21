# Laravel Nova Default Fields (Snipped - No Install)

Standard fields for each resource.  
No installation necessary.  
Tested with Nova 4.

## Instruction

Add following content to the Nova base resource (`app/Nova/Resources/Resource.php`):

```php
     /**
     * Get the fields that are available for the given request.
     *
     * @param NovaRequest  $request
     * @return FieldCollection
     */
    public function availableFields(NovaRequest $request): FieldCollection
    {
        $method = $this->fieldsMethod($request);
        $fields = array_merge($this->{$method}($request), $this->defaultFields($request));

        return FieldCollection::make(array_values($this->filter($fields)));
    }

    /**
     * Get the fields that are available for the given request.
     *
     * @param NovaRequest $request
     * @param  array  $methods
     * @return FieldCollection
     */
    public function buildAvailableFields(NovaRequest $request, array $methods): FieldCollection
    {
        $fields = collect([
            method_exists($this, 'fields') ? array_merge($this->fields($request), $this->defaultFields($request)) : [],
        ]);

        collect($methods)
            ->filter(function ($method) {
                return $method != 'fields' && method_exists($this, $method);
            })->each(function ($method) use ($request, $fields) {
                $fields->push([$this->{$method}($request)]);
            });

        return FieldCollection::make(array_values($this->filter($fields->flatten()->all())));
    }

    /**
     * Default fields for every resource
     *
     * @param NovaRequest $request
     * @return array
     */
    protected function defaultFields(NovaRequest $request): array
    {
        return [];
    }


```

### Define Default Fields

You can define the default fields in the `defaultFields` method

#### Example 1

```php
    protected function defaultFields(NovaRequest $request): array
    {
        return [
            DateTime::make(__('Created at'), 'created_at')
                ->sortable()->filterable()->exceptOnForms(),
            DateTime::make(__('Updated at'), 'updated_at')
                ->sortable()->filterable()->exceptOnForms(),
        ];
    }
```

#### Example 2

With options `$showCreatedAtField`, `showCreatedAtField()`, `$showUpdatedAtField` & `showUpdatedAtField()` and check if required colum exist `static::$model::CREATED_AT` &  `static::$model::UPDATED_AT`

```php
    /**
     * Show in this resource the DateTime field `created_at`
     *
     * @var bool
     */
    public bool $showCreatedAtField = true;
    public function showCreatedAtField(NovaRequest $request): bool
    {
        return $this->showCreatedAtField;
    }

    /**
     * Show in this resource the DateTime field `updated_at`
     *
     * @var bool
     */
    public bool $showUpdatedAtField = true;
    public function showUpdatedAtField(NovaRequest $request): bool
    {
        return $this->showUpdatedAtField;
    }

    protected function defaultFields(NovaRequest $request): array
    {
        $fields = [];

        if ($this->showCreatedAtField($request) && static::$model::CREATED_AT) {
            $fields = array_merge($fields, [
                DateTime::make(__('Created at'), 'created_at')
                    ->sortable()->filterable()->exceptOnForms()
            ]);
        }

        if ($this->showUpdatedAtField($request) && static::$model::UPDATED_AT) {
            $fields = array_merge($fields, [
                DateTime::make(__('Updated at'), 'updated_at')
                    ->sortable()->filterable()->exceptOnForms()
            ]);
        }

        /*
         * For SoftDeletes
         * */
        if ($request->input('trashed')) {
            $fields = array_merge($fields, [
                DateTime::make(__('Deleted at'), 'deleted_at')
                    ->sortable()->filterable()->exceptOnForms()
            ]);
        }

        /*
         * For spatie/laravel-activitylog
         * */
        if (method_exists(static::$model, 'getLogName')) {
            $fields = array_merge($fields, [
                MorphMany::make(__('Activities'), 'activities', Activity::class)
            ]);
        }

        return $fields;
    }
```
---
[![More Laravel Nova Packages](https://raw.githubusercontent.com/Muetze42/asset-repo/main/svg/more-laravel-nova-packages.svg)](https://huth.it/nova-packages)

[![Stand With Ukraine](https://raw.githubusercontent.com/vshymanskyy/StandWithUkraine/main/banner2-direct.svg)](https://vshymanskyy.github.io/StandWithUkraine/)
[![Woman. Life. Freedom.](https://raw.githubusercontent.com/Muetze42/Muetze42/2033b219c6cce0cb656c34da5246434c27919bcd/files/iran-banner-big.svg)](https://linktr.ee/CurrentPetitionsFreeIran)
