# 附录 I. 控制器提取后的代码

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
14 $controller = new \Controller\ArticlesPage();
15
16 /* CONTROLLER */
17
18 $response = $controller->__invoke(
19 $request,
20 $response,
21 $user,
22 $article_transactions
23 );
24
25 /* FINISHED */
26
27 $response->send();
28 ?>
**classes/Controller/ArticlesPage.php**
1 <?php
2 namespace Controller;
3
4 use Domain\Articles\ArticleTransactions;
5 use Mlaphp\Request;
6 use Mlaphp\Response;
7 use User;
8
9 class ArticlesPage
10 {
11 public function __construct()
12 {
13 }
14
15 public function __invoke(
16 Request $request,
17 Response $response,
18 User $user,
19 ArticleTransactions $article_transactions
20 ) {
21 $user_id = $user->getId();
22
23 $id = isset($request->get['id'])
24 ? $request->get['id']
25 : 0;
26
27 if ($id) {
28 $article_transactions->updateExistingArticle(
29 $user_id,
30 $request->post
31 );
32 } else {
33 $article_transactions->submitNewArticle(
34 $user_id,
35 $request->post
36 );
37 }
38
39 $response->setView('articles.html.php');
40 $response->setVars(array(
41 'id' => $id,
42 'failure' => $article_transactions->getFailure(),
43 'input' => $article_transactions->getInput(),
44 'action' => $request->server['PHP_SELF'],
45 ));
46
47 return $response;
48 }
49 }
50 ?>
```
