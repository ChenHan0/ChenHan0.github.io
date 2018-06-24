---
title: "【转移】Unity法线翻转(flip normal)"
categories:
    - U3D
tag:
    - flip normal 
    - U3D
last_modified_at: 2018-01-12T14:15:00+08:00
---

##### 首发于简书，2016.08.01 12:53

直接上干货  
  
```cs  
        Vector3[] normals = line.GetComponent<MeshFilter>().mesh.normals;
        for (int i = 0; i < normals.Length; i++) {
            normals[i] = -normals[i];
        }
        line.GetComponent<MeshFilter>().mesh.normals = normals;

        int[] triangles = line.GetComponent<MeshFilter>().mesh.triangles;
        for (int i = 0; i < triangles.Length; i += 3) {
            int t = triangles[i];
            triangles[i] = triangles[i + 2];
            triangles[i + 2] = t;
        }
        line.GetComponent<MeshFilter>().mesh.triangles = triangles;
```    
---  
  
<!-- more -->  
  
刚开始想到做法线翻转，只想到了直接把法线取负值，就是第一段代码  

```cs  
        Vector3[] normals = line.GetComponent<MeshFilter>().mesh.normals;
        for (int i = 0; i < normals.Length; i++) {
            normals[i] = -normals[i];
        }
        line.GetComponent<MeshFilter>().mesh.normals = normals;
```  

结果是这样子：  

![](/assets/images/flip-normal-img-1.png)
  
就是一个没有任何光线信息的样子（~~纯黑~~）  

然后呢，我就一直百度，以求找到解决办法，百度了一个晚上，都是说，“为什么不到3dmax/MAYA中翻转法线呢”，~~你大爷的！~~  

然后转战Google，5分钟后解决问题= =  

```  
        int[] triangles = line.GetComponent<MeshFilter>().mesh.triangles;
        for (int i = 0; i < triangles.Length; i += 3) {
            int t = triangles[i];
            triangles[i] = triangles[i + 2];
            triangles[i + 2] = t;
        }
        line.GetComponent<MeshFilter>().mesh.triangles = triangles;
```    

加上了第二段代码之后，是这样子的：

![](/assets/images/flip-normal-img-2.png)  
  
（是的，我做的是卡通渲染的描边，没使用shader）  
  
`Mesh.vertices`中，保存的是图形的顶点信息。  
`Mesh.triangles`中，保存的是对应于`Mesh.vertices`的顶点的索引。就是一个三角形在渲染中的三个顶点的顺序，所以`Mesh.triangles`的长度应该是3的倍数（`Mesh.triangles`的类型为int[]）  
那为什么要交换第一点跟第三点的位置呢？  
  
---  
  
![](/assets/images/flip-normal-img-3.png)  
  
假如现在一个三角形是由P<sub>0</sub>、P<sub>1</sub>、P<sub>2</sub>，三个点组成的一个三角形。  
那么，他的绘制的顺序应该是这样子的：  
  
![](/assets/images/flip-normal-img-4.png)  

呈现一个逆时针的样子。图形学中（前几天看蓝宝书看到的，忘了是OpenGL中的还是说图形学中都是这样，请指正。~~Ps. 应该是逆时针吧~~）将拥有逆时针环绕的多边形为正面。  
即上面这个三角形为正面  
若我们从屏幕后面那个方向看这个三角形的话，那你看到的是他的背面。
当我们交换三角形的第一点跟第三点后，他的渲染顺序将变成这样子：  

![](/assets/images/flip-normal-img-5.png)
  
变成了从P<sub>2</sub>到P<sub>1</sub>再到P<sub>0</sub>的这么一个顺序。即这个三角形现在是一个顺时针环绕，我们看到的这个面，是他的背面。  
在Unity中，默认的渲染是不会渲染背面的。  
这样子就会出现刚才的这个效果  
  
![](/assets/images/flip-normal-img-6.png)  
  
---  
  
这是本人写的第一个文章，有什么不足之处，请指正。
写本文的目的在于，苦苦寻找解决解决方法的您不会像我一样，找了一晚上的百度，啥都没找到。（~~Ps.这样应该能百度得到吧~~）