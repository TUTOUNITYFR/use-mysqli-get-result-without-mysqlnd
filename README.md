# Use mysqli_stmt get_result() without mysqlnd driver
This repository contains a sample code to use [**mysqli_stmt::get_result()**](https://www.php.net/manual/en/mysqli-stmt.get-result.php) method in PHP without having to install [mysqlnd driver](https://www.php.net/manual/en/book.mysqlnd.php) on your server.

Sample code used is based on Sophivorus answer in [this thread](https://stackoverflow.com/questions/10752815/mysqli-get-result-alternative/30551477#30551477).

## Intro
With mysqli_stmt you can use both **bind_result()** and **get_result()** methods. The problem with using bind_result is that it requires you to specify the columns that will be used in your statement. Which means that you will have to assign a lot of variables and it makes keeping track of large queries difficult.

If you have the [**MySQL Native Driver (mysqlnd)**](https://www.php.net/manual/en/book.mysqlnd.php) installed on your server, all you need to do is use get_result(). But what if you do not want (or can not) install mysqlnd on your server ?

In this case, you can use the sample code bellow which *"simulate"* the mysqli_stmt::get_result() method. This code simply loops through the results of your query and builds an associative array.

## Sample code
Add this method in your .php file :
```php
function get_result(\mysqli_stmt $statement)
{
    $result = array();
    $statement->store_result();
    for ($i = 0; $i < $statement->num_rows; $i++)
    {
        $metadata = $statement->result_metadata();
        $params = array();
        while ($field = $metadata->fetch_field())
        {
            $params[] = &$result[$i][$field->name];
        }
        call_user_func_array(array($statement, 'bind_result'), $params);
        $statement->fetch();
    }
    return $result;
}
```

Then, to retrieve data from your database with a prepared statement, use :
```php
$stmt = $database->prepare('SELECT x FROM y WHERE z = ?');
$stmt->bind_param('s', $z);
$stmt->execute();
$result = get_result($stmt);
while ( $data = array_shift($result) ) {
    echo $data["x"];
}

// Or return the object
// return array_shift($result);

```
