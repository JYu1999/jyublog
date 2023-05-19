---
title: "Laravel Backpack 初學者之路-05"
date: 2023-05-19T19:36:26+08:00
draft: false
tags:
  - Laravel
  - Laravel Backpack
categories:
  - laravel_backpack
image: "0.png"
keywords:
  - Laravel
  - Laravel Backpack
  - CMS
  - Admin
---

上一篇在 UpdateOperation 中，有一個 `update()` 的 method，這個 method 又去 call 了一個叫做 `update()`的 method：

```php
public function update()  
{  
    $this->crud->hasAccessOrFail('update');  
  
    // execute the FormRequest authorization and validation, if one is required  
    $request = $this->crud->validateRequest();  
  
    // register any Model Events defined on fields  
    $this->crud->registerFieldEvents();  
  
    // update the row in the db  
    $item = $this->crud->update(  
        $request->get($this->crud->model->getKeyName()),  
        $this->crud->getStrippedSaveRequest($request)  
    );  
    $this->data['entry'] = $this->crud->entry = $item;  
  
    // show a success message  
    \Alert::success(trans('backpack::crud.update_success'))->flash();  
  
    // save the redirect choice for next time  
    $this->crud->setSaveAction();  
  
    return $this->crud->performSaveAction($item->getKey());  
}
```

這一篇主要就是要探討後面這個 `update()`。

這邊來仔細看一下 `$this->crud->update()` 做了些什麼事情。

`update()` 的具體實現如下：
```php
public function update($id, $input)  
{  
    $item = $this->model->findOrFail($id);  
  
    [$directInputs, $relationInputs] = $this->splitInputIntoDirectAndRelations($input);  
    $updated = $item->update($directInputs);  
    $this->createRelationsForItem($item, $relationInputs);  
  
    return $item;  
}
```

`findOrFail()` 很明顯就是要給定 id，找這個 id 的相關 data。

而 `splitInputIntoDirectAndRelations($input)` 是要把 input 分為 `DirectInputs` 和 `RelationDetails` 。

然後下面會把分出來的 `$directInputs` 存進資料庫裡面，

最後會用 `createRelationsForItem() `幫 `$item` 和所提供的 model 建立關聯。

處理關聯的主要是 `createRelationsForItem()` ，所以這裡繼續來 trace。

---

## createRelationsForItem()
這個 method 需要兩個輸入：
* `$item`：要創建或更新 relationship 的 model instance
* `$formattedRelations`: 包含關聯資訊的 array，是在上面用 `getRelationDetailsFromInput()` 拿到的。

這個 method 會先檢查  `formmattedRelations` 是否為空，為空就不用創建 relationships。否則會遍歷 `$formattedRelations[]`，根據不同的關聯類型來創建或更新相應的 relationships。

主要有三種 relationship 類型：
### HasOne 或 MorphOne
使用 `createUpdateOrDeleteOneToOneRelation()` 方法創建或更新一對一關聯。

```php
private function createUpdateOrDeleteOneToOneRelation($relation, $relationMethod, $relationDetails)  
{   
        $relationMethodValue = $relationDetails['values'][$relationMethod];  
  
        
        if (is_null($relationMethodValue) && $relationDetails['entity'] === $relationMethod) {  
            $relation->delete();  
  
            return null;  
        }  
  

        if (gettype($relationMethodValue) == 'array' && is_multidimensional_array($relationMethodValue)) {  
            $relationMethodValue = $relationMethodValue[0];  
        }  
    }  
    // saving process  
    $input = $relationMethodValue ?? $relationDetails['values'];  
    [$directInputs, $relationInputs] = $this->splitInputIntoDirectAndRelations($input, $relationDetails, $relationMethod);  
  
    $item = $relation->updateOrCreate([], $directInputs);  
  
    $this->createRelationsForItem($item, $relationInputs);  
  
    return $item;  
}
```

根據官方的說法，這個 method 會根據 values 不同，而有不同的處理方式：
* (A) `['number' => 1315, 'name' => 'Something']` ：如果使用 text, number, etc... field
* (B) `['slug' => null]` ：如果需要清除 `slug` 和其相關的 entry。
* (C) `['passport' => [['number' => 1314, 'name' => 'Something']]]` ：如果值是透過 repeatable field 傳遞
* (D) `['passport' => null]`：如果值被從 repeatable field 中刪除。

首先，`array_key_exists($relationMethod, $relationDetails['values'])` 會檢查是不是 (C) or (D)。如果是 (D) 的話，會把 relation 刪掉並回傳 null。

我還蠻疑惑檢查機制的，於是把 `values[]` 印出來（雖然這個是多對多）：
```php
  "values" => array:1 [▼
    "articles" => array:1 [▼
      0 => "2"
    ]
  ]
```

把檢查機制印出來就一目了然，上面的檢查是在看 `model` 裡面的 relationship method 是不是 array 的其中一個 key。

最後，會把這次更新做保存。`$directInputs` 會 call `updateOrCreate()` 存進資料庫。然後會把 relation input 傳進 `createRelationsForItem()` 檢查看還有沒有 relation 需要處理。

### HasMany 或 MorphMany
根據 `$relationValues` 的結構，使用 `attachManyRelation()` 建立已存在的 model instance 之間的 relation，或者使用 `createManyEntries()` 方法創建新的 model instance 並建立 relation。

```php
$relationValues = $relationDetails['values'][$relationMethod];  

if ($relationValues === null || ! is_multidimensional_array($relationValues)) {  
    $this->attachManyRelation($item, $relation, $relationDetails, $relationValues);  
} else {  
    $this->createManyEntries($item, $relation, $relationMethod, $relationDetails);  
}  
break;
```

#### attachManyRelation()
首先先來看 `attachManyRelation()`：
```php
    private function attachManyRelation($item, $relation, $relationDetails, $relationValues)
    {
        $modelInstance = $relation->getRelated();
        $relationForeignKey = $relation->getForeignKeyName();
        $relationLocalKey = $relation->getLocalKeyName();

        if (empty($relationValues)) {

            return $this->handleManyRelationItemRemoval($modelInstance, $relation, $relationDetails, $relationForeignKey);
        }

        $toUpdate[$relationForeignKey] = $item->{$relationLocalKey};

        if ($relationDetails['relation_type'] === 'MorphMany') {
            $toUpdate[$relation->getQualifiedMorphType()] = $relation->getMorphClass();
        }

        $modelInstance->whereIn($modelInstance->getKeyName(), $relationValues)
            ->update($toUpdate);


        $removedEntries = $modelInstance->whereNotIn($modelInstance->getKeyName(), $relationValues)
                            ->where($relationForeignKey, $item->{$relationLocalKey});


        if ($relationDetails['relation_type'] === 'MorphMany') {
            $removedEntries->where($relation->getQualifiedMorphType(), $relation->getMorphClass());
        }

        return $this->handleManyRelationItemRemoval($modelInstance, $removedEntries, $relationDetails, $relationForeignKey);
    }

```

在 hasmany/morphmany 中，並沒有 「同步(sync)」這個 method 可以用。但當我們在使用 selectable elments 我們需要同步的功能，所以需要寫一些程式去模擬同步，讓使用者可以去新增/刪除那些沒有在 list 裡面的 entries。

這個 method 會先檢查 `relationValues` 是否為空，如果為空代表 selection 被清除掉了，所以程式要清除 all related values。會利用 `handleManyRelationItemRemoval()` 這個 method 來做這件事情，至於這個 method 的實作在下方討論。

在下面需要把新的 value 加進 relation 裡面，如果是 HasMany 的話，那只需要 update foreign key；如果是 MorphMany 的話，就需要 update foreign key & morphs key。

下面會從 model instance 找所有不在 `$relationValues` 的紀錄，也就是在當前 relation 中已經被刪除的紀錄。

如果 relation type 是 morphmany，還要確保有 match 到 morph type。

最後，會把已刪除的 entry(`$removedEntries`) 透過 `handleManyRelationItemRemoval()` 刪除。

#### handleManyRelationItemRemoval()
```php
private function handleManyRelationItemRemoval($modelInstance, $removedEntries, $relationDetails, $relationForeignKey)
    {
        $relationColumnIsNullable = $modelInstance->isColumnNullable($relationForeignKey);
        $forceDelete = $relationDetails['force_delete'] ?? false;
        $fallbackId = $relationDetails['fallback_id'] ?? false;

        
        if ($fallbackId) {
            return $removedEntries->update([$relationForeignKey => $fallbackId]);
        }

        
        if ($forceDelete) {
            return $removedEntries->lazy()->each->delete();
        }

        
        $dbColumnDefault = $modelInstance->getDbColumnDefault($relationForeignKey);

        
        if (! $relationColumnIsNullable && $dbColumnDefault === null) {
            return $removedEntries->lazy()->each->delete();
        }

        
        return $removedEntries->update([$relationForeignKey => $dbColumnDefault]);
    }
```

這個 method 主要處理 remove 和特定 relation 相關的 entry。

一開始會先看有沒有指定 `$fallbackId`，如果有的話只需要改一下 foreignkey 即可。

或是如果有設定 `force_delete` 是 true 的話，就不考慮 foreign key 是否是 nullable，直接刪除所有被移除的 entries。

如果沒有設定設定 `force_delete` 的話，則會檢查 foreign key 是否可為空，以及 database 的設定：
* if column 可為空：就設為 default value 就好（沒有的話就設 null）
* if column 不可為空：會把 entry 從資料庫刪掉，避免資料庫報錯。

#### createManyEntries()
```php
    private function createManyEntries($entry, $relation, $relationMethod, $relationDetails)
    {
        $items = $relationDetails['values'][$relationMethod];

        $relatedModelLocalKey = $relation->getRelated()->getKeyName();

        $relatedItemsSent = [];

        foreach ($items as $item) {
            [$directInputs, $relationInputs] = $this->splitInputIntoDirectAndRelations($item, $relationDetails, $relationMethod);
            $relatedModelLocalKeyValue = $item[$relatedModelLocalKey] ?? null;

            $item = $entry->{$relationMethod}()->updateOrCreate([$relatedModelLocalKey => $relatedModelLocalKeyValue], $directInputs);

            $relatedItemsSent[] = $item->{$relatedModelLocalKey};

            $this->createRelationsForItem($item, $relationInputs);
        }

        if (! empty($relatedItemsSent)) {
            $entry->{$relationMethod}()->whereNotIn($relatedModelLocalKey, $relatedItemsSent)->lazy()->each->delete();
        }
    }
```

首先，會先從 `$relationDetails` 中獲取和 `$relationMethod`相關的所有 entries。

這裡會拿到 model 的 key 之後，宣告一個空陣列 `$relatedItemsSent[]`，但這邊還看不出來這個是要幹嘛。

接著，會對所有拿出來的 `$items`作遍歷，在遍歷的過程中，一樣會先把 direct input 和 relation input 分開，並或去 related model 的 local key value。

接著，使用model 的 instance，找到匹配的 entry，根據 local key 更新輸入值或創建新的 entry。
已經存進去的會放到 `relatedItemSent[]　`， 避免重複發送。最後會幫 entry 創建 relation。

在這個 method 的最後，會去檢查 `$relatedItemsSent`是否為空。如果是不是空的話，會使用已經發送的 items 和  database 的 local key 作匹配，並刪除沒有在兩者交集裡面的值。

---

### BelongsToMany 或 MorphToMany
獲取關聯值並確保它們不為空，然後根據值的結構創建或同步多對多關聯。

```php
                case 'BelongsToMany':
                case 'MorphToMany':
                    $values = $relationDetails['values'][$relationMethod] ?? [];
                    $values = is_string($values) ? json_decode($values, true) : $values;

                    $values = $values ?? [];

                    $relationValues = [];

                    if (is_array($values) && is_multidimensional_array($values)) {
                        foreach ($values as $value) {
                            if (isset($value[$relationMethod])) {
                                $relationValues[$value[$relationMethod]] = Arr::except($value, $relationMethod);
                            }
                        }
                    }
                    
                    if (empty($relationValues)) {
                        $relationValues = array_values($values);
                    }

                    $item->{$relationMethod}()->sync($relationValues);
                    break;
```

首先，會從 `$relationDetails` 中獲取該 relation 的 value，並確保其為 array。

`json_decode()` 可能會回傳 null，因此要確保 `$values` 不為 null。

在下面 initialize 一個新的 empty array `$relationValues`。接著，如果 `$values` 是 multidimensional_array 的話，就會遍歷 `$values`，並獲取每個值中的 `$relationMethod` 作為 key，其餘的值作為 `$relationValues` 。在這裡是使用 `Arr::except($value, $relationMethod)` ，代表是要從 `$value[]` 中移除 `$relationMethod` 這個 key 及其對應的值。

如果沒有 relation data，且 `values` 是一個單維陣列，則為把 `$values` 的值作為 `$relationValues`。

最後，使用`$item->{$relationMethod}()->sync($relationValues);` 同步 relation value。`sync()` 會把多對多關聯表的現有 relation，更新為這裡所提供的 `$relationValues`，並在需要時創建新的 relation or 刪除不再存在的 relation。
