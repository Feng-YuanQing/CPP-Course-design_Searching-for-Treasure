# 探寻宝藏
***************
## 动态规划解决算法，DFS、opencv绘制迷宫
---------------------------------------
```C++
  #include <iostream>
  #include <opencv2/opencv.hpp>
  #include "opencv2/imgproc.hpp"
  #include "opencv2/highgui.hpp"
  #include <stack>
  #include <string>

  int Data[52][52], dp[52][52][52][52];//k数据组数，数据m行n列，dp宝藏总价值

  using namespace std;
  using namespace cv;

  class M_Point
  {
  public:
    int row;
    int col;
    bool operator == (const M_Point &p)
    {
      return ((this->row == p.row) && (this->col == p.col));
    }
    friend ostream& operator << (ostream &os, M_Point &point) //ostream, out stream 输出流，常用于 << 的重载， 作为某个类的友元函数出现。
    {
      os << '(' << point.row << ',' << point.col << ')';
      return os;
    }
  };

  class Grid
  {
  public:
    M_Point place;
    int direction = 0;
    bool isWay;
    bool isPassed = false;
  };

  class Maze
  {
  public:
    int row;
    int col;
    M_Point start;
    M_Point end;
    Grid** map = NULL;

    Maze(int m, int n) :row(m), col(n)
    {
      if (m == 0 || n == 0)
        return;
      map = new Grid*[m + 1];
      for (int i = 0; i < m; ++i)
      {
        map[i] = new Grid[n];
      }
    }

    ~Maze()
    {
      if (map == NULL || row == 0 || col == 0)
        return;
      else
      {
        for (int i = 0; i < row; ++i)
        {
          delete[](map[i]);
        }
        delete[](map);
      }
    }
  };

  const Scalar wall_color(200, 200, 200);
  const Scalar pass_color(100, 150, 100);
  const Scalar start_end_color(150, 100, 100);
  const int gap = 2;
  const int delay = 200;

  void drawBlock(Mat &img, int row, int col, int height, int width, Scalar color, const int Data[][52])
  {
    //初始化点坐标
    Point p1 = Point(col * width + gap, row * height + gap);
    Point p2 = Point((col + 1) * width - gap, (row + 1) * height - gap);
    //rectangle( CvArr* img, CvPoint pt1, CvPoint pt2, CvScalar color,int thickness = 1, int line_type = 8, int shift = 0 );
    //img
    //	图像.
    //pt1
    //	矩形的一个顶点。
    //pt2
    //	矩形对角线上的另一个顶点
    //color
    //	线条颜色(RGB) 或亮度（灰度图像 ）(grayscale image）。
    //thickness
    //		组成矩形的线条的粗细程度。取负值时（如 CV_FILLED）函数绘制填充了色彩的矩形。
    //line_type
    //		线条的类型。见cvLine的描述
    //shift
    //		坐标点的小数点位数。
    rectangle(img, p1, p2, color, -1);
    char str[10];
    sprintf(str, "%d", Data[row + 1][col + 1]);
    // cvPoint 为起笔的x，y坐标
    putText(img, str, cvPoint(col * width + gap, (row + 1) * height - gap), CV_FONT_HERSHEY_COMPLEX, 0.65, CV_RGB(0, 0, 255));//在图片中输出字符
    //
    //	cv::Mat& img, // 待绘制的图像,
    //	const string& text, // 待绘制的文字
    //	cv::Point origin, // 文本
