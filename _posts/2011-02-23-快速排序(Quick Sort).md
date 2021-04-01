---
layout: post
book: true
background-image: http://ot1cc1u9t.bkt.clouddn.com/17-7-16/91630214.jpg
category: 算法/数据结构
title: 快速排序-Quick Sort.
tags:
- 算法
- 排序
---
快速排序-Quick Sort

原理
======
通过一趟排序将要排序的数据分割成独立的两部分
其中一部分的所有数据都比另外一部分的所有数据都要小
然后再按此方法对这两部分数据分别进行快速排序。

示例
======
```
class Sort
{

    public function testSort()
    {
        echo '快排排序<br />';
        $list = [9, 5, 2, 7, 3, 21, 22, 4, 88, 6, 17, 20];

        print_r($list);
        $this->sort($list, 0, count($list) - 1);

        print_r($list);

        return '';
    }

    // 通过一趟排序将要排序的数据分割成独立的两部分
    // 其中一部分的所有数据都比另外一部分的所有数据都要小
    // 然后再按此方法对这两部分数据分别进行快速排序
    private function sort(&$arr, $l, $u)
    {
        if ($l >= $u) {
            echo "$l -- $u 局部排序完毕!<br />=====================<br/>";
            return;
        }
        echo "给 $l -- $u 这一段排序!<br />---------------------<br/>";

        $temp = $arr[$l];
        $i = $l;
        $j = $u + 1;

        // 执行完后.
        while (true) {
            // 从前往后遍历一次排序区间.找到第一个比区间最左侧值大的值
            do {
                $i++;
            } while ($i <= $u && $arr[$i] < $temp);

            // 从后往前,找到一个比区间最左侧值小的值
            do {
                $j--;
            } while ($arr[$j] > $temp);

            // 如果从右往前找到的第一个比 $i 大的值在 $i 的左侧.说明划分区间已经完成
            if ($i > $j) {
                echo '区间划分已经完成!<br />';
                break;
            }
            echo '从左到右找到了第一个比 $arr[$i] ' . " = $temp 大的值: $arr[$i] <br />";
            echo '从右到左找到了第一个比 $arr[$i] ' . " = $temp 小的值: $arr[$j] <br />";
            // 交换两个值.
            // 目的是通过这个循环,把大于排序区间第一个值和小于的,全部分成两边
            // $j 左侧全部是比 $arr[$l] 小的值;右侧全部是比它大的值
            // $arr[$l] 是排序区间的第一个值
            $this->swap($arr, $i, $j);
        }
        print_r($arr);
        // 将 $l 和 $j 对应的值调换
        // $j 其实是临界值.它左边的全比 $arr[$l] 小,右边的全比 $arr[$l] 大
        // 将这两个值调换后. $arr[$j] 就是标准的临界值了.它的左侧值全部比它自己小,右侧值全部比它自己大
        $this->swap($arr, $l, $j);
        print_r($arr);
        // 递归调用自己.把左、右两边分别再排序
        $this->sort($arr, $l, $j - 1);
        $this->sort($arr, $j + 1, $u);

    }

    private function swap(&$arr, $a, $b)
    {
        echo "交换第 $arr[$a] 和 $arr[$b] 的值!<br />";
        $temp = $arr[$a];

        $arr[$a] = $arr[$b];
        $arr[$b] = $temp;

    }

}
```