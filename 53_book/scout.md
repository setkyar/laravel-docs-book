# Laravel Scout

- [မိတ်ဆက်](#introduction)
- [Installation](#installation)
    - [Queueing](#queueing)
    - [Driver Prerequisites](#driver-prerequisites)
- [Configuration](#configuration)
    - [Configuring Model Indexes](#configuring-model-indexes)
    - [Configuring Searchable Data](#configuring-searchable-data)
- [Indexing](#indexing)
    - [Batch Import](#batch-import)
    - [Adding Records](#adding-records)
    - [Updating Records](#updating-records)
    - [Removing Records](#removing-records)
    - [Pausing Indexing](#pausing-indexing)
- [Searching](#searching)
    - [Where Clauses](#where-clauses)
    - [Pagination](#pagination)

<a name="introduction"></a>
## မိတ်ဆက်

[Eloquent models](/docs/{{version}}/eloquent) အတွက် full-text search ကို Laravel  Scout ကလွယ်ကူတဲ့ driver based solution အဖြစ်ပြုလုပ်ပေးထားပါတယ်။ Model observers တွေကိုသုံးပြီးတော့ search indexes အဖြစ် Eloquent records တွေနဲ့အလိုအလျောက် sync လုပ်ပေးမှာဖြစ်ပါတယ်။

လက်ရှိမှာတော့ Scout ကို [Algolia](https://www.algolia.com/) driver နဲ့ ship လုပ်ပေးမှာဖြစ်ပါတယ်၊ ဒါပေမယ့် custom drivers တွေပြုလုပ်ရတာကလွယ်ကူတော့ Scout ကို extend လုပ်ပြီးတော့ကိုယ်ပိုင် search implementations တွေလည်းပြုလုပ်နိုင်ပါတယ်။

<a name="installation"></a>
## Installation

ပထမဆုံး Scout ကို Composer package manager ကနေ install လုပ်လိုက်ပါ -

    composer require laravel/scout

ပြီးရင်တော့ `ScoutServiceProvider` ကို `config/app.php` array configuration file မှာထည့်လိုက်ပါ -

    Laravel\Scout\ScoutServiceProvider::class,

Scout service provider ကို register လုပ်ပြီးပြီဆိုရင် Scout configuration ကို `vendor:publish` Artisan command run ပြီးတော့ publish လုပ်လိုက်ပါ။ ဒီ publish command run လိုက်တာနဲ့ `config` directory မှာ `scout.php` configuration file ကို publish လုပ်သွားမှာဖြစ်ပါတယ် -

    php artisan vendor:publish

နောက်ဆုံးအဆင့်အနေနဲ့ကတော့ `Laravel\Scout\Searchable` trait ကိုကိုယ်ရှာစေချင်တဲ့ Model မှာထည့်ပါ။ အဲ့ဒီ့ trait က model နဲ့ search driver ကြားမှာ sync လုပ်အောင် model observer တစ်ခု register လုပ်ပါလိမ့်မယ်။

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;
    }

<a name="queueing"></a>
### Queueing

Scout ကိုအတင်းအကျပ်မသုံးမပြုခိုင်းပေမယ့် အသုံးပြုမယ်ဆိုရင် [queue driver](/docs/{{version}}/queues) ကိုအရင်ဆုံး configure လုပ်ဖို့တိုက်တွန်းချင်ပါတယ်။ Queue worker ကို run ထားရင် Socut ကို model information တွေနဲ့ search indexes တွေကို sync လုပ်ရာသင့် application web interface ကပိုမိုမြန်ဆန်စေပါလိမ့်မယ်...

Queue driver ကို configure လုပ်ပြီးသွားပြီဆိုရင် `config/scout.php` configuration ထဲက `queue` configuration ကို `true` ပြောင်းပေးပါ...

    'queue' => true,

<a name="driver-prerequisites"></a>
### Driver Prerequisites

#### Algolia

Algolia driver ကိုအသုံးပြုတဲ့အခါမှာ Algolia `id` နဲ့ `secret` credentials တွေကို `config/scout.php`မှာ configure လုပ်သင့်ပါတယ်။ သင့် credentials တွေကို configure လုပ်ပြီးသွားပြီဆိုရင် Algolia PHP SDK ကို Composer package manager ကနေ install လုပ်ဖို့လိုပါတယ်...

    composer require algolia/algoliasearch-client-php

<a name="configuration"></a>
## Configuration

<a name="configuring-model-indexes"></a>
### Configuring Model Indexes

Each Eloquent model is synced with a given search "index", which contains all of the searchable records for that model. In other words, you can think of each index like a MySQL table. By default, each model will be persisted to an index matching the model's typical "table" name. Typically, this is the plural form of the model name; however, you are free to customize the model's index by overriding the `searchableAs` method on the model:

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * Get the index name for the model.
         *
         * @return string
         */
        public function searchableAs()
        {
            return 'posts_index';
        }
    }

<a name="configuring-searchable-data"></a>
### Configuring Searchable Data

By default, the entire `toArray` form of a given model will be persisted to its search index. If you would like to customize the data that is synchronized to the search index, you may override the `toSearchableArray` method on the model:

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * Get the indexable data array for the model.
         *
         * @return array
         */
        public function toSearchableArray()
        {
            $array = $this->toArray();

            // Customize array...

            return $array;
        }
    }

<a name="indexing"></a>
## Indexing

<a name="batch-import"></a>
### Batch Import

If you are installing Scout into an existing project, you may already have database records you need to import into your search driver. Scout provides an `import` Artisan command that you may use to import all of your existing records into your search indexes:

    php artisan scout:import "App\Post"

<a name="adding-records"></a>
### Adding Records

Once you have added the `Laravel\Scout\Searchable` trait to a model, all you need to do is `save` a model instance and it will automatically be added to your search index. If you have configured Scout to [use queues](#queueing) this operation will be performed in the background by your queue worker:

    $order = new App\Order;

    // ...

    $order->save();

#### Adding Via Query

If you would like to add a collection of models to your search index via an Eloquent query, you may chain the `searchable` method onto an Eloquent query. The `searchable` method will [chunk the results](/docs/{{version}}/eloquent#chunking-results) of the query and add the records to your search index. Again, if you have configured Scout to use queues, all of the chunks will be added in the background by your queue workers:

    // Adding via Eloquent query...
    App\Order::where('price', '>', 100)->searchable();

    // You may also add records via relationships...
    $user->orders()->searchable();

    // You may also add records via collections...
    $orders->searchable();

The `searchable` method can be considered an "upsert" operation. In other words, if the model record is already in your index, it will be updated. If it does not exist in the search index, it will be added to the index.

<a name="updating-records"></a>
### Updating Records

To update a searchable model, you only need to update the model instance's properties and `save` the model to your database. Scout will automatically persist the changes to your search index:

    $order = App\Order::find(1);

    // Update the order...

    $order->save();

You may also use the `searchable` method on an Eloquent query to update a collection of models. If the models do not exist in your search index, they will be created:

    // Updating via Eloquent query...
    App\Order::where('price', '>', 100)->searchable();

    // You may also update via relationships...
    $user->orders()->searchable();

    // You may also update via collections...
    $orders->searchable();

<a name="removing-records"></a>
### Removing Records

To remove a record from your index, simply `delete` the model from the database. This form of removal is even compatible with [soft deleted](/docs/{{version}}/eloquent#soft-deleting) models:

    $order = App\Order::find(1);

    $order->delete();

If you do not want to retrieve the model before deleting the record, you may use the `unsearchable` method on an Eloquent query instance or collection:

    // Removing via Eloquent query...
    App\Order::where('price', '>', 100)->unsearchable();

    // You may also remove via relationships...
    $user->orders()->unsearchable();

    // You may also remove via collections...
    $orders->unsearchable();

<a name="pausing-indexing"></a>
### Pausing Indexing

Sometimes you may need to perform a batch of Eloquent operations on a model without syncing the model data to your search index. You may do this using the `withoutSyncingToSearch` method. This method accepts a single callback which will be immediately executed. Any model operations that occur within the callback will not be synced to the model's index:

    App\Order::withoutSyncingToSearch(function () {
        // Perform model actions...
    });

<a name="searching"></a>
## Searching

You may begin searching a model using the `search` method. The search method accepts a single string that will be used to search your models. You should then chain the `get` method onto the search query to retrieve the Eloquent models that match the given search query:

    $orders = App\Order::search('Star Trek')->get();

Since Scout searches return a collection of Eloquent models, you may even return the results directly from a route or controller and they will automatically be converted to JSON:

    use Illuminate\Http\Request;

    Route::get('/search', function (Request $request) {
        return App\Order::search($request->search)->get();
    });

<a name="where-clauses"></a>
### Where Clauses

Scout allows you to add simple "where" clauses to your search queries. Currently, these clauses only support basic numeric equality checks, and are primarily useful for scoping search queries by a tenant ID. Since a search index is not a relational database, more advanced "where" clauses are not currently supported:

    $orders = App\Order::search('Star Trek')->where('user_id', 1)->get();

<a name="pagination"></a>
### Pagination

In addition to retrieving a collection of models, you may paginate your search results using the `paginate` method. This method will return a `Paginator` instance just as if you had [paginated a traditional Eloquent query](/docs/{{version}}/pagination):

    $orders = App\Order::search('Star Trek')->paginate();

You may specify how many models to retrieve per page by passing the amount as the first argument to the `paginate` method:

    $orders = App\Order::search('Star Trek')->paginate(15);

Once you have retrieved the results, you may display the results and render the page links using [Blade](/docs/{{version}}/blade) just as if you had paginated a traditional Eloquent query:

    <div class="container">
        @foreach ($orders as $order)
            {{ $order->price }}
        @endforeach
    </div>

    {{ $orders->links() }}
