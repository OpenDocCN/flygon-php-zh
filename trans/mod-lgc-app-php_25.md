# 附录 H. 控制器重新排列后的代码

```php
**articles.php**
1 <?php
2 require "includes/setup.php";
3
4 /* DEPENDENCY */
5
6 $db = new Database($db_host, $db_user, $db_pass);
7 $articles_gateway = new ArticlesGateway($db);
8 $users_gateway = new UsersGateway($db);
9 $article_transactions = new ArticleTransactions(
10 $articles_gateway,
11 $users_gateway
12 );
13 $response = new \Mlaphp\Response('/path/to/app/views');
14
15 /* CONTROLLER */
16
17 $user_id = $user->getId();
18
19 $id = isset($_GET['id']) ? $_GET['id'] : 0;
20 if ($id) {
21 $article_transactions->updateExistingArticle($user_id, $_POST);
22 } else {
23 $article_transactions->submitNewArticle($user_id, $_POST);
24 }
25
26 $response->setView('articles.html.php');
27 $response->setVars(array(
28 'id' => $id,
29 'failure' => $article_transactions->getFailure(),
30 'input' => $article_transactions->getInput(),
31 'action' => $_SERVER['PHP_SELF'],
32 ));
33
34 /* FINISHED */
35
36 $response->send();
37 ?>
```
