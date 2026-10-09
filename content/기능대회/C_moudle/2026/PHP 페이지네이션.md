# PHP 페이징 (10개씩, 이전/다음)

## 1. 현재 페이지 구하기

URL

```url
board.php?page=3
```

PHP

```php
$page = $_GET['page'] ?? 1;
```

- `?page=3`이면 `$page = 3`
- `?page`가 없으면 `$page = 1`

## 2. 한 페이지에 보여줄 개수

```php
$perPage = 10;
```

- 한 페이지당 게시글 10개씩 표시

## 3. DB에서 가져올 시작 번호 계산

```php
$start = ($page - 1) * $perPage;
```

| 페이지 | 계산 | 시작 번호 |
| --- | --- | --- |
| 1 | (1-1)×10 | 0 |
| 2 | (2-1)×10 | 10 |
| 3 | (3-1)×10 | 20 |
| 4 | (4-1)×10 | 30 |

## 4. 게시글 10개 가져오기

```php
$list = db::fetchAll("
  SELECT *
  FROM board
  ORDER BY idx DESC
  LIMIT $start, $perPage
");
```

- 예: 2페이지는 `LIMIT 10, 10`이므로 11~20번째 글 조회

## 5. 전체 게시글 개수 구하기

```php
$total = db::fetch("
  SELECT COUNT(*) cnt
  FROM board
")->cnt;
```

- 예: `$total = 35;`

## 6. 마지막 페이지 구하기

```php
$maxPage = ceil($total / $perPage);
```

- 예: `ceil(35 / 10)`은 `4`

## 7. 이전 버튼

```php
<?php if ($page > 1): ?>
  <a href="?page=<?= $page - 1 ?>">
    이전
  </a>
<?php endif; ?>
```

| 현재 | 이동 |
| --- | --- |
| 1 | 없음 |
| 2 | 1 |
| 3 | 2 |
| 4 | 3 |

## 8. 다음 버튼

```php
<?php if ($page < $maxPage): ?>
  <a href="?page=<?= $page + 1 ?>">
    다음
  </a>
<?php endif; ?>
```

| 현재 | 이동 |
| --- | --- |
| 1 | 2 |
| 2 | 3 |
| 3 | 4 |
| 4 | 없음 |

## 9. 페이지 번호 출력

```php
<?php for ($i = 1; $i <= $maxPage; $i++): ?>
  <a href="?page=<?= $i ?>">
    <?= $i ?>
  </a>
<?php endfor; ?>
```

## 10. 현재 페이지 강조

```php
<?php for ($i = 1; $i <= $maxPage; $i++): ?>
  <a
    href="?page=<?= $i ?>"
    <?= $i == $page ? 'style="font-weight:bold"' : '' ?>
  >
    <?= $i ?>
  </a>
<?php endfor; ?>
```

## 전체 코드

```php
<?php
$perPage = 10;
$page = $_GET['page'] ?? 1;

$start = ($page - 1) * $perPage;

$list = db::fetchAll("
  SELECT *
  FROM board
  ORDER BY idx DESC
  LIMIT $start, $perPage
");

$total = db::fetch("
  SELECT COUNT(*) cnt
  FROM board
")->cnt;

$maxPage = ceil($total / $perPage);
?>

<?php foreach ($list as $item): ?>
  <div><?= $item->title ?></div>
<?php endforeach; ?>

<div>
  <?php if ($page > 1): ?>
    <a href="?page=<?= $page - 1 ?>">이전</a>
  <?php endif; ?>

  <?php for ($i = 1; $i <= $maxPage; $i++): ?>
    <a href="?page=<?= $i ?>">
      <?= $i ?>
    </a>
  <?php endfor; ?>

  <?php if ($page < $maxPage): ?>
    <a href="?page=<?= $page + 1 ?>">다음</a>
  <?php endif; ?>
</div>
```
