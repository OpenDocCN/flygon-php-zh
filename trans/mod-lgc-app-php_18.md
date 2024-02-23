# 附录 A. 典型的传统页面脚本

```php
<?php
2 include("common/db_include.php");
3 include("common/functions.inc");
4 include("theme/leftnav.php");
5 include("theme/header.php");
6
7 define("SEARCHNUM", 10);
8
9 function letter_links()
10 {
11 global $p, $letter;
12 $lettersArray = array(
13 '0-9', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I',
14 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S',
15 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'
16 );
17 foreach ($lettersArray as $let) {
18 if ($letter == $let)
19 echo $let.' ';
20 else
21 echo '<a class="letters" '
22 . 'href="letter.php?p='
23 . $p
24 . '&letter='
25 . $let
26 . '">'
27 . $let
28 . '</a> ';
29 }
30 }
31
32 $page = ($page) ? $page : 0;
33
34 if (!empty($p) && $p!="all" && $p!="none") {
35 $where = "`foo` LIKE '%$p%'";
36 } else {
37 $where = "1";
38 }
39
40 if ($p=="hand") {
41 $where = "`foo` LIKE '%type1%'"
42 . " OR `foo` LIKE '%type2%'"
43 . " OR `foo` LIKE '%type3%'";
44 }
45
46 $where .= " AND `bar`='1'";
47 if ($s) {
48 $s = str_replace(" ", "%", $s);
49 $s = str_replace("'", "", $s);
50 $s = str_replace(";", "", $s);
51 $where .= " AND (`baz` LIKE '%$s%')";
52 $orderby = "ORDER BY `baz` ASC";
53 } elseif ($letter!="none" && $letter) {
54 $where .= " AND (`baz` LIKE '$letter%'"
55 . " OR `baz` LIKE 'The $letter%')";
56 $orderby = "ORDER BY `baz` ASC";
57 } else {
58 $orderby = "ORDER BY `item_date` DESC";
59 }
60 $query = mysql_query(
61 "SELECT * FROM `items` WHERE $where $orderby
62 LIMIT $page,".SEARCHNUM;
63 );
64 $count = db_count("items", $where);
65 ?>
66
67 <td align="middle" width="480" valign="top">
68 <img border="0" width="480" height="30"
69 src="http://example.com/images/example1.gif">
70 <table border="0" cellspacing="0" width="480"
71 cellpadding="0" bgcolor="#000000">
72 <tr>
73 <td colspan="2" width="480" height="50">
74 <img border="0"
75 src="http://example.com/images/example2.gif">
76 </td>
77 </tr>
78 <tr>
79 <td width="120" align="right" nowrap>
80 <img border="0"
81 src="http://example.com/images/example3.gif">
82 </td>
83 <td width="360" align="right" nowrap>
84 <div class="letter"><?php letter_links(); ?></div>
85 </td>
86 </tr>
87 </table>
88
89 <form name="search" enctype="multipart/form-data"
90 action="search.php" method="POST" margin="0"
91 style="margin: 0px;">
92 <table border="0" style="border-collapse: collapse"
93 width="480" cellpadding="0">
94 <tr>
95 <td align="center" width="140">
96 <input type="text" name="s" size="22"
97 class="user_search" title="enter your search..."
98 value="<?php
99 echo $s
100 ? $s
101 : "enter your search..."
102 ;
103 ?>" onFocus=" enable(this); "
104 onBlur=" disable(this); ">
105 </td>
106 <td align="center" width="70">
107 <input type="image" name="submit"
108 src="http://example.com/images/user_search.gif"
109 width="66" height="17">
110 </td>
111 <td align="right" width="135">
112 <img border="0"
113 src="http://example.com/images/list_foo.gif"
114 width="120" height="26">
115 </td>
116 <td align="center" width="135">
117 <select size="1" name="p" onChange="submit();">
118 <?php
119 if ($p) {
120 ${$p} = 'selected="selected"';
121 }
122 foreach ($foos as $key => $value) {
123 echo '<option value="'
124 . $key
125 . '" '
126 . ${$key}
127 . '>'
128 . $value
129 . '</option>';
130 }
131 ?>
132 </select>
133 </td>
134 </tr>
135 </table>
136 <?php if ($letter) {
137 echo '<input type="hidden" name="letter" '
138 . 'value="' . $letter . '">';
139 } ?>
140 </form>
141
142 <table border="0" cellspacing="0" width="480"
143 cellpadding="0" style="border-style: solid; border-color:
144 #606875; border-width: 1px 1px 0px 1px;">
145 <tr>
146 <td>
147 <div class="nav"><?php
148 $pagecount = ceil(($count / SEARCHNUM));
149 $currpage = ($page / SEARCHNUM) + 1;
150 if ($pagecount)
151 echo ($page + 1)
152 . " to "
153 . min(($page + SEARCHNUM), $count)
154 . " of $count";
155 ?></div>
156 </td>
157 <td align="right">
158 <div class="nav"><?php
159 unset($getstring);
160 if ($_POST) {
161 foreach ($_POST as $key => $val) {
162 if ($key != "page") {
163 $getstring .= "&$key=$val";
164 }
165 }
166 }
167 if ($_GET) {
168 foreach ($_GET as $key => $val) {
169 if ($key != "page") {
170 $getstring .= "&$key=$val";
171 }
172 }
173 }
174
175 if (!$pagecount) {
176 echo "No results found!";
177 } else {
178 if ($page >= (3*SEARCHNUM)) {
179 $firstlink = " | <a class=\"searchresults\"
180 href=\"?page=0$getstring\">1</a>";
181 if ($page >= (4*SEARCHNUM)) {
182 $firstlink .= " ... ";
183 }
184 }
185
186 if ($page >= (2*SEARCHNUM)) {
187 $prevpages = " | <a class=\"searchresults\""
188 . " href=\"?page="
189 . ($page - (2*SEARCHNUM))
190 . "$getstring\">"
191 . ($currpage - 2)
192 ."</a>";
193 }
194
195 if ($page >= SEARCHNUM) {
196 $prevpages .= " | <a class=\"searchresults\""
197 . " href=\"?page="
198 . ($page - SEARCHNUM)
199 . "$getstring\">"
200 . ($currpage - 1)
201 . "</a>";
202 }
203
204 if ($page==0) {
205 $prevlink = "&laquo; Previous";
206 } else {
207 $prevnum = $page - SEARCHNUM;
208 $prevlink = "<a class=\"searchresults\""
209 . " href=\"?page=$prevnum$getstring\">"
210 . "&laquo; Previous</a>";
211 }
212
213 if ($currpage==$pagecount) {
214 $nextlink = "Next &raquo;";
215 } else {
216 $nextnum = $page + SEARCHNUM;
217 $nextlink = "<a class=\"searchresults\""
218 . " href=\"?page=$nextnum$getstring\">"
219 . "Next &raquo;</a>";
220 }
221
222 if ($page < (($pagecount - 1) * SEARCHNUM))
223 $nextpages = " | <a class=\"searchresults\""
224 . " href=\"?page="
225 . ($page + SEARCHNUM)
226 . "$getstring\">"
227 . ($currpage + 1)
228 . "</a>";
229
230 if ($page < (($pagecount - 2)*SEARCHNUM)) {
231 $nextpages .= " | <a class=\"searchresults\""
232 . " href=\"?page="
233 . ($page + (2*SEARCHNUM))
234 . "$getstring\">"
235 . ($currpage + 2)
236 . "</a>";
237 }
238
239 if ($page < (($pagecount - 3)*SEARCHNUM)) {
240 if ($page < (($pagecount - 4)*SEARCHNUM))
241 $lastlink = " ... of ";
242 else
243 $lastlink = " | ";
244 $lastlink .= "<a class=\"searchresults\""
245 . href=\"?page="
246 . (($pagecount - 1)*SEARCHNUM)
247 . "$getstring\">"
248 . $pagecount
249 . "</a>";
250 }
251
252 $pagenums = " | <b>$currpage</b>";
253 echo $prevlink
254 . $firstlink
255 . $prevpages
256 . $pagenums
257 . $nextpages
258 . $lastlink
259 . ' | '
260 . $nextlink;
261 }
262 ?></div>
263 </td>
264 </tr>
265 </table>
266
267 <table border="0" cellspacing="0" width="100%"
268 cellpadding="0" style="border-style: solid; border-color:
269 #606875; border-width: 0px 1px 0px 1px;">
270
271 <?php while($item = mysql_fetch_array($query)) {
272
273 $links = get_links(
274 $item[id],
275 $item[filename],
276 $item[fileinfotext]
277 );
278
279 $dls = get_dls($item['id']);
280
281 echo '
282 <tr>
283 <td class="bg'.(($ii % 2) ? 1 : 2).'" align="center">
284
285 <div style="margin:10px">
286 <table border="0" style="border-collapse:
287 collapse" width="458" id="table5" cellpadding="0">
288 <tr>
289 <td rowspan="3" width="188">
290 <table border="0" cellpadding="0"
291 cellspacing="0" width="174">
292 <tr>
293 <td colspan="4">
294 <img border="0"
295 src="http://www.example.com/common/'
296 .$item[thumbnail].'"
297 width="178" height="74"
298 class="media_img">
299 </td>
300 </tr>
301 <tr>
302 <td style="border-color: #565656;
303 border-style: solid; border-width: 0px
304 0px 1px 1px;" width="18">
305 <a target="_blank"
306 href="'.$links[0][link].'"
307 '.$links[0][addlink].'>
308 <img border="0"
309 src="http://example.com/images/'
310 .$links[0][type].'.gif"
311 width="14" height="14"
312 hspace="3" vspace="3">
313 </a>
314 </td>
315 <td style="border-color: #565656;
316 border-style: solid; border-width: 0px
317 0px 1px 0px;" align="left" width="71">
318 <a target="_blank"
319 href="'.$links[0][link].'"
320 class="media_download_link"
321 '.$links[0][addlink].'>'
322 .(round($links[0][filesize]
323 / 104858) / 10).' MB</a>
324 </td>
325 <td style="border-color: #565656;
326 border-style: solid; border-width: 0px
327 0px 1px 0px;" width="18">
328 '.(($links[1][type]) ? '<a
329 target="_blank"
330 href="'.$links[1][link].'"
331 '.$links[1][addlink].'><img
332 border="0"
333 src="http://example.com/images/'
334 .$links[1][type].'.gif"
335 width="14" height="14" hspace="3"
336 vspace="3">
337 </td>
338 <td style="border-color: #565656;
r339 border-style: solid; border-width: 0px
340 1px 1px 0px;" align="left" width="71">
341 <a target="_blank"
342 href="'.$links[1][link].'"
343 class="media_download_link"
344 '.$links[1][addlink].'>'
345 .(round($links[1][filesize]
346 / 104858) / 10).' MB</a>' :
347 '&nbsp;</td><td>&nbsp;').'
348 </td>
349 </tr>
350 </table>
351 </td>
352 <td width="270" valign="bottom">
353 <div class="list_title">
354 <a
355 href="page.php?id='.$item[rel_id].'"
356 class="list_title_link">'.$item[baz].'</a>
357 </div>
358 </td>
359 </tr>
360 <tr>
361 <td align="left" width="270">
362 <div class="media_text">
363 '.$item[description].'
364 </div>
365 </td>
366 </tr>
367 <tr>
368 <td align="left" width="270">
369 <div class="media_downloads">'
370 .number_format($dls)
371 .' Downloads
372 </div>
373 </td>
374 </tr>
375 </table>
376 </div>
377 </td>
378 </tr>';
379 $ii++;
380 } ?>
381 </table>
382
383 <table border="0" cellspacing="0" width="480"
384 cellpadding="0" style="border-style: solid; border-color:
385 #606875; border-width: 0px 1px 1px 1px;">
386 <tr>
387 <td>
388 <div class="nav"><?php
389 if ($pagecount)
390 echo ($page + 1)
391 . " to "
392 . min(($page + SEARCHNUM), $count)
393 . " of $count";
394 ?></div>
395 </td>
396 <td align="right">
397 <div class="nav"><?php
398 if (!$pagecount) {
399 echo "No search results found!";
400 } else {
401 echo $prevlink
402 . $firstlink
403 . $prevpages
404 . $pagenums
405 . $nextpages
406 . $lastlink
407 . ' | '
408 . $nextlink;
409 }
410 ?></div>
411 </td>
412 </tr>
413 </table>
414 </td>
415
416 <?php include("theme/footer.php"); ?>
```
