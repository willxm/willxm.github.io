---
title: K-Means algorithm implementation in Go
date: 2018-09-02 21:51:43
tags: k-means
intro : two-dimension K-Means
comments: false
---

# K-Means

## 介绍
K-Means算法属于无监督学习的一种分类算法，K表示分组的数量。K-Means是基于距离的聚类算法，变量之间的距离越近，就表示相似度越大。

## 步骤

1. 输入一组离散的数据P
2. 根据K值生成在给定数据范围之内的K个随机点，叫做质心
3. 计算P中每个点分别到K个质心的距离（下文采用欧式距离），将这个点归于最近的质心
4. 重新计算质心点，```x=(x1+x2+...+xn)/n,y=(y1+y2+...+yn)/2```
5. 重复上面的步骤3-4，直到结果收敛。


## 实现

```go
type Point struct {
	X float64
	Y float64
}

//EuclideanDistance 计算欧式距离
func EuclideanDistance(p1, p2 Point) float64 {
	return math.Sqrt(math.Pow(p1.X-p2.X, 2) + math.Pow(p1.Y-p2.Y, 2))
}
// Centroid 计算质心
func Centroid(points []Point) Point {
	l := len(points)
	sx, sy := 0.0, 0.0
	for i := 0; i < l; i++ {
		sx += points[i].X
		sy += points[i].Y
	}
	return Point{
		X: sx / float64(l),
		Y: sy / float64(l),
	}
}
```

```go
// KMeans ...
func KMeans(points []kit.Point, k, iter int) map[kit.Point][]kit.Point {
	minX, maxX, minY, maxY := kit.DataRange(kit.ToSlice(points))
	//Init k random points
	var randomPoints []kit.Point
	for i := 0; i < k; i++ {
		x := kit.Random(minX, maxX)
		y := kit.Random(minY, maxY)
		randomPoints = append(randomPoints, kit.Point{X: x, Y: y})
	}
	// iteration
	kps := randomPoints
	for i := 0; i < iter; i++ {
		kps = kmeansStep(points, kps)
	}
	res := kmeansClass(points, kps)
	log.Printf("%+v", res)
	return res
}

func kmeansStep(points, kps []kit.Point) []kit.Point {
	kmap := make(map[kit.Point][]kit.Point, len(kps))
	for _, p := range points {
		minDist := float64(math.MaxFloat64)
		point := kit.Point{}
		for _, kp := range kps {
            //把距离小的点 append 到 map 中 进行分类
			dist := kit.EuclideanDistance(p, kp)
			if minDist > dist {
				minDist = dist
				point.X = kp.X
				point.Y = kp.Y
			}
		}
		kmap[point] = append(kmap[point], p)
	}
	res := make([]kit.Point, len(kps))
	i := 0
	for _, v := range kmap {
		res[i] = kit.Centroid(v)
		i++
	}
	return res
}
```

## 测试
INPUT

X | Y
--|--
0 | 0
1 | 2
3 | 1
8 | 8
9 | 10
10 | 7

K = 2


OUTPUT
```go
{X:9 Y:8.333333333333334}:[{X:8 Y:8} {X:9 Y:10} {X:10 Y:7}
{X:1.3333333333333333 Y:1}:[{X:0 Y:0} {X:1 Y:2} {X:3 Y:1}
```

