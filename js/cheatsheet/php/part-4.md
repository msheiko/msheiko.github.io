```php
<?php

class Product
{
    /**
     * @var int
     */
    public $id;
    /**
     * @var string
     */
    public $name;
    /**
     * @var float
     */
    public $price;
    /**
     * @var string
     */
    public $image;
    /**
     * @var string
     */
    public $description;
    /**
     * @var string
     */
    public $created_date;
    /**
     * @var string
     */
    public $updated_date;

    /**
     * Product constructor.
     * @param string $name
     * @param float $price
     * @param string $image
     * @param string $description
     */
    public function __construct(string $name, float $price, string $image, string $description)
    {
        $this->name = $name;
        $this->price = $price;
        $this->image = $image;
        $this->description = $description;
    }

}

//create table products
//(
//    id int auto_increment,
//	name varchar(100) null,
//	price float not null,
//	image varchar(255) null,
//	description text null,
//	created_date datetime null,
//	updated_date datetime null,
//	constraint products_pk
//		primary key (id)
//);
```

```php
<?php
interface ProductRepositoryInterface
{
    public function find($id):Product;
    public function save(Product $product):bool;
    public function remove(Product $product):bool;
    public function update(Product $product):bool;
}
```

```php
<?php
require_once "Product.php";
require_once "ProductRepositoryInterface.php";
class MysqlProductRepository implements ProductRepositoryInterface
{
    protected $db;
    /**
     * MysqlProductRepository constructor.
     */
    public function __construct()
    {
        $dsn = "mysql:host=mysql;dbname=database;charset=UTF8";
        $this->db = new PDO($dsn,"user","userpassword");
    }

    public function find($id): Product
    {
        $stmt = $this->db->prepare('SELECT * FROM `database`.`products` WHERE `id`=:id');
        $stmt->bindParam(":id",$id,PDO::PARAM_INT);
        $stmt->setFetchMode(PDO::FETCH_CLASS | PDO::FETCH_PROPS_LATE, Product::class, ["", 0, "", ""]);
        $stmt->execute();
        return $stmt->fetch();
    }

    public function save(Product $product): bool
    {
        $sql = "INSERT INTO `database`.`products` (`name`, `price`, `image`, `description`, `created_date`, `updated_date`) VALUES 
                                                   (:name, :price, :image, :description, :created_date, :updated_date)";
        $stmt = $this->db->prepare($sql);
        unset($product->id);
        $product->created_date = date("Y-m-d H:i:s");
        $product->updated_date = null;
        $params = (array) $product;
        return $stmt->execute($params);
    }

    public function remove(Product $product): bool
    {
        // TODO: Implement remove() method.
    }

    public function update(Product $product): bool
    {
        // TODO: Implement update() method.
    }
}
```

#  ?????????????????????? ????????

?????? ???????? ???? ???????????????? ???????????????????????? ??????????????????

?? PHP ???????????????????????? ???????? ???????????????????????? ?????? ?????????????? ???????? ??????????????, ?? ???????????????? ???????????????????????? ???????????? ?????????????????? ?? ???????????????????? ?????? ???????????????? ???????????????? ???????????????????????? ?????????????????? ????????, ?????????? ?????? ???????????? ?? ??????????????:

- ???????????????? ???????? ?????????? ?????????? ?????????? ?? ?????????????????????? ????????????????/??????????????????/?????????????????????? PHP ?????? ????????????????????.
- ?????????????????????? ?????????????????? ???????????????????? (?????? ????????????????????) ?????? ????_??????????_??????????????_????????, ?????????? ?????????????????? ???????????? ???????????????? ?? ???????????????? ???????????????????? ?????????????????? ????????.
  
    ?????????? ?????????????????????? ???????? ????????????????????????????????????.

```
"autoload": {
    "classmap": [
      "path/to/FirstClass.php",
      "path/to/SecondClass.php"
    ]
  }
```

```
require_once __DIR__ . "/../vendor/autoload.php";
```

```
composer dump-autoload -o
```

```
{
    "name": "test/test",
    "type": "project",
    "require": {},
    "autoload": {
      "classmap": [
        "Domain/Product.php",
        "Repository/MysqlProductRepository.php",
        "Repository/ProductRepositoryInterface.php"
      ]
    }
}
```

# Json

[JsonSerializable](https://www.php.net/manual/ru/jsonserializable.jsonserialize.php)

```php
    public function jsonSerialize()
    {
        return get_object_vars($this);
        return [
            "name" => $this->name,
            "price" => $this->price;
        ]
    }
```

# ???????????? ?? ?????????????? 

??????????

```php
$file = fopen('counter.txt', 'r');
```


- r ??? ???????????????? ?????????? ???????????? ?????? ????????????.
- r+ - ???????????????? ?????????? ???????????????????????? ???? ???????????? ?? ????????????.
- w ??? ???????????????? ???????????? ?????????????? ??????????. ???????? ???? ???????????? ???????????? ?????? ???????????????????? ?????????? ????????, ???? ???? ????????????????????????.
- w+ - ???????????????????? r+, ???????????? ???????? ???? ???????????? ???????????? ?????? ?????????? ????????????????????, ?????? ???????????????????? ??????????????????.
- a ??? ?????????????????? ???????????????????????? ???????? ??  ???????????? ????????????, ?????? ???????? ?????????????????? ???????????????????? ????  ?????????????????? ???????? ?????????? (???? ?????????? ??????????).
- a+ - ?????????????????? ???????? ?? ???????????? ???????????? ?? ???????????? ?????? ???????? ?????????????????? ???????????????????? ???? ?????????????????? ???????? ?????????? (???? ?????????? ??????????). ???????????????????? ?????????? ???? ??????????????????.


[php-fig](https://github.com/php-fig/fig-standards/tree/master/accepted)

### ???????????????? ?????????? ?? psr

```php
    public function emergency($message, array $context = array())
    {

        $result = fwrite($this->logFile,date("Y.m.d h:m:s")." - ".$message."\n");
        if ($result) echo '???????????? ?? ???????? ?????????????? ????????????????.';
        else echo '???????????? ?????? ???????????? ?? ????????.';
    }
```

# ?????????????????? 

```
$logger = new Logger();
$main = new Main($logger);
$main->sayHell();
```

```
$container = new Container();
$container->set(ProductRepositoryInterface::class, new MysqlProductRepository());
```



# Eloquent

```php
<?php

namespace App\DB;

use App\Domain\User;
use Illuminate\Database\Capsule\Manager as Capsule;
use Illuminate\Database\Schema\Blueprint;

class DBManager
{
    private $capsule;
    public function __construct()
    {
        $this->capsule = new Capsule;
        $this->capsule->addConnection([
            "driver" => "mysql",
            "host" =>"mysql",
            "database" => "database",
            "username" => "user",
            "password" => "userpassword"
        ]);

        $this->capsule->setAsGlobal();
        $this->capsule->bootEloquent();
    }

    public function createUsersTable()
    {
        $this->capsule::schema()->create('users', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('email')->unique();
            $table->string('password');
            $table->string('userimage')->nullable();
            $table->rememberToken();
            $table->timestamps();

        });
    }

    public function createUser()
    {
        User::Create(['name' => "Ahmed Khan",    'email' => "ahmed.khan@lbs.com",    'password' => password_hash("ahmedkhan",PASSWORD_BCRYPT), ]);
        $user = new User();
    }
}
```

```php
<?php

namespace App\Domain;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Query\Builder;

/**
 * Class User
 * @package App\Domain
 * /**
 * @method static Builder where($column, $operator = null, $value = null, $boolean = 'and')
 * @method static Builder create(array $attributes = [])
 * @method public Builder update(array $values)
 */
class User extends Model
{
    /**
     * @var array
     */
    protected $fillable = [

        'name', 'email', 'password','userimage'

    ];

    /**
     * @var array
     */
    protected $hidden = [

        'password', 'remember_token',

    ];
}
```

[????????????](https://medium.com/@kshitij206/use-eloquent-without-laravel-7e1c73d79977)

```php
    public function createCommentTable()
    {
        $this->capsule::schema()->create('comments', function (Blueprint $table) {
            $table->increments('id');
            $table->string('body');
            $table->timestamps();

            $table->integer('user_id')->unsigned();

            $table->foreign('user_id')->references('id')->on('users')->onDelete('cascade');

        });
    }
```

```php
    public function comments()
    {
        return $this->hasMany('App\Domain\Comment');
    }
```

```php
$user = User::findOrFail(1);

var_dump($user->comments->count());
```