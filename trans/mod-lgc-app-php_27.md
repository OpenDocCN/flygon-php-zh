# 附录 J. 控制器依赖注入后的代码

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
14 $controller = new \Controller\ArticlesPage(
15 $request,
16 $response,
17 $user,
18 $article_transactions
19 );
20
21 /* CONTROLLER */
22
23 $response = $controller->__invoke();
24
25 /* FINISHED */
26
27 $response->send();
28 ?>
```

```php
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
11 protected $user;
12
13 protected $article_transactions;
14
15 protected $request;
16
17 protected $response;
18
19 public function __construct(
20 Request $request,
21 Response $response,
22 User $user,
23 ArticleTransactions $article_transactions
24 ) {
25 $this->user = $user;
26 $this->article_transactions = $article_transactions;
27 $this->request = $request;
28 $this->response = $response;
29 }
30
31 public function __invoke()
32 {
33 $user_id = $this->user->getId();
34
35 $id = isset($this->request->get['id'])
36 ? $this->request->get['id']
37 : 0;
38
39 if ($id) {
40 $article_transactions->updateExistingArticle(
41 $user_id,
42 $this->request->post
43 );
44 } else {
Appendix J: Code After Controller Dependency Injection 217
45 $article_transactions->submitNewArticle(
46 $user_id,
47 $this->request->post
48 );
49 }
50
51 $this->response->setView('articles.html.php');
52 $this->response->setVars(array(
53 'id' => $id,
54 'failure' => $this->article_transactions->getFailure(),
55 'input' => $this->article_transactions->getInput(),
56 'action' => $this->request->server['PHP_SELF'],
57 ));
58
59 return $this->response;
60 }
61 }
62 ?>
```
