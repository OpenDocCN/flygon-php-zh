# 附录 G. 响应视图文件后的代码

```php
**articles.php**
1 <?php
2 require "includes/setup.php";
3
4 $user_id = $user->getId();
5
6 $db = new Database($db_host, $db_user, $db_pass);
7 $articles_gateway = new ArticlesGateway($db);
8 $users_gateway = new UsersGateway($db);
9 $article_transactions = new ArticleTransactions(
10 $articles_gateway,
11 $users_gateway
12 );
13
14 $id = isset($_GET['id']) ? $_GET['id'] : 0;
15 if ($id) {
16 $article_transactions->updateExistingArticle($user_id, $_POST);
17 } else {
18 $article_transactions->submitNewArticle($user_id, $_POST);
19 }
20
21 $response = new \Mlaphp\Response('/path/to/app/views');
22 $response->setView('articles.html.php');
23 $response->setVars(array(
24 'id' => $id,
25 'failure' => $article_transactions->getFailure(),
26 'input' => $article_transactions->getInput(),
27 'action' => $_SERVER['PHP_SELF'],
28 ));
29 $response->send();
30 ?>
**views/articles.html.php**
1 <?php
2 $current_page = 'articles';
3
4 include "header.php";
5
6 if ($id) {
7 $page_title = "Edit An Article";
8 } else {
9 $page_title = "Submit An Article";
10 }
11 ?>
12
13 <h1><?php echo $page_title ?></h1><?php
14
15 if ($failure) {
16 echo "<h2>Failure</h2>";
17 echo "<p>We could not save the article.<br />";
18 foreach ($failure as $failure_text) {
19 echo $this->esc($failure_text) . "<br />";
20 }
21 echo "</p>";
22 } else {
23 echo "
24 <h2>Success</h2>
25 <p>We saved the article.</p>
26 ";
27 }
28 ?>
29
30 <form method="POST" action="<?php echo $this->esc($action) ?>">
31
32 <input type="hidden" name="id" value="<?php echo $this->esc($id) ?>" />
33
34 <h3>Title</h3>
35 <input type="text" name="title" value="<?php
36 echo $this->esc($input['title'])
37 ?>" size="100">
38
39
40 <h3>Article</h3>
41 <textarea name="body" cols="80" rows="30"><?php
42 echo stripslashes($this->esc($input['body']))
43 ?></textarea>
44
45 <h3>Ratings</h3>
46 <p>How many rated reviews do you want?</p>
47 <select name='max_ratings'>
48 <?php for ($i = 1; $i <= 10; $i ++) {
49 $i = $this->esc($i);
50 echo "<option value='$i' ";
51 if ($input['max_ratings'] == $i) {
52 echo 'selected="selected"';
53 }
54 echo ">$i</option>\n";
55 } ?>
56 </select>
57
58 <p>How many credits will you give for each rating?</p>
59 <input type='text' name='credits_per_rating' value='<?php
60 echo $this->esc($input['credits_per_rating']);
61 ?>' size='5' />
62
63 <h3>Notes for Reviewers</h3>
64 <input type="text" name="notes" value="<?php
65 echo $this->esc($input['notes'])
66 ?>" size="100">
67 <label><input type="checkbox" name="ready" <?php
68 echo ($input['ready']) ? 'checked="checked"' : '';
69 ?> /> This article is ready to be rated.</label>
70
71 <p align="center">
72 <input type="submit" value="Save" name="submit">
73 </p>
74
75 </form>
76
77 <?php
78 include "footer.php";
79 ?>
```
