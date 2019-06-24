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
    //	cv::Point origin, // 文本框的左下角
    //	int fontFace, // 字体 (如cv::FONT_HERSHEY_PLAIN)
    //	double fontScale, // 尺寸因子，值越大文字越大
    //	cv::Scalar color, // 线条的颜色（RGB）
    //	int thickness = 1, // 线条宽度
    //	int lineType = 8, // 线型（4邻域或8邻域，默认8邻域）
    //	bool bottomLeftOrigin = false // true='origin at lower left'
    //	);
  }

  void drawMaze(Mat &img, const Maze &maze, const int Data[][52])
  {
    int height = img.rows / maze.row;
    int width = img.cols / maze.col;
    for (int i = 0; i < maze.row; ++i)
    {
      for (int j = 0; j < maze.col; ++j)
      {
        if (maze.map[i][j].place == maze.start || maze.map[i][j].place == maze.end)
          drawBlock(img, i, j, height, width, start_end_color, Data);
        else
          drawBlock(img, i, j, height, width, wall_color, Data);
      }
    }
    imshow("maze", img);
    waitKey(0);
  }

  //m表示行，n表示列
  stack<Grid*> maze_solution(const Maze &maze, Mat &img, int flag, const int Data[][52])
  {
    int height = img.rows / maze.row;
    int width = img.cols / maze.col;

    Grid** map = maze.map;
    stack<Grid*> way;

    //起点或终点不在迷宫内
    if (maze.start.row >= maze.row || maze.start.col >= maze.col
      || maze.end.row >= maze.row || maze.end.col >= maze.col)
      return way;

    Grid *current = &map[maze.start.row][maze.start.col];
    while (1)
    {
      map[maze.start.row][maze.start.col].isPassed = false;
      map[maze.end.row][maze.end.col].isPassed = false;
      do
      {
        if (flag == 1)
        {
          //当前的位置是可行路径且之前未经过
          if (current->isWay && !current->isPassed)
          {
            current->isPassed = true;
            way.push(current);

            //如果到达终点，则返回该条路径
            if (current->place == maze.end)

              break;
            else
            {
              if (!(current->place == maze.start))
                drawBlock(img, current->place.row, current->place.col, height, width, pass_color, Data);
              imshow("maze", img);
              waitKey(delay);
              if (current->place.row != maze.end.row)
                //默认每次先探索下边的方向
                current = &map[current->place.row + 1][current->place.col];
              else
                //到达下边界后向右
                current = &map[current->place.row][current->place.col + 1];
            }
          }
          else
          {
            current = way.top(); current = &map[current->place.row][current->place.col + 1];
          }
        }
        if (flag == 2)
        {
          //当前的位置是可行路径且之前未经过
          if (current->isWay && !current->isPassed)
          {
            current->isPassed = true;
            way.push(current);

            //如果到达终点，则返回该条路径
            if (current->place == maze.start)
            {
              waitKey(0);
              return way;
            }
            else
            {
              if (!(current->place == maze.end))
                drawBlock(img, current->place.row, current->place.col, height, width, pass_color, Data);
              imshow("maze", img);
              waitKey(delay);
              if (current->place.row != maze.start.row)
                //默认每次先探索上边的方向
                current = &map[current->place.row - 1][current->place.col];
              else
                //到达上边界后向左
                current = &map[current->place.row][current->place.col - 1];
            }
          }
          else
          {
            current = way.top();
            current = &map[current->place.row][current->place.col - 1];
          }
        }
      } while (!way.empty());
      if (flag != 2) { flag++;waitKey(600); }
      else break;
    }
    return way;
  }

  int max(int a, int b, int c, int d)//重载max函数
  {
    return max(max(a, b), max(c, d));
  }

  int main()
  {
    int k;
    cin >> k;
    while (k--)
    {
      int m, n;    //m,n 表示迷宫的规模，m行n列
      cin >> m >> n;
      fill(Data[0], Data[0] + 52 * 52, 0);
      fill(dp[0][0][0], dp[0][0][0] + 52 * 52 * 52 * 52, 0);    //使用fill两个数组初始化为0； 
      for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
          cin >> Data[i][j];
      Maze maze(m, n);
      maze.start.row = 0; maze.start.col = 0;
      maze.end.row = m - 1; maze.end.col = n - 1;
      for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
          maze.map[i][j].isWay = 0;
      int maxdp;
      for (int i = 2; i <= m; i++)
        for (int j = 1; j <= n; j++)
          for (int x = 1; x <= i; x++)
            for (int y = 2; y <= n; y++)
            {
              if (i == x && j == y || i + j != x + y) continue;
              maxdp = max(dp[i - 1][j][x - 1][y], dp[i - 1][j][x][y - 1],
                dp[i][j - 1][x - 1][y], dp[i][j - 1][x][y - 1]);
              dp[i][j][x][y] = maxdp + Data[i][j] + Data[x][y];   //找到其中最大值并加上 (i,j) (x,y)两个位置本身含有的值 
              if (x > 1 && j > 1 && y > 1 && i > 1)
              {
                if (maxdp == dp[i - 1][j][x - 1][y]) { maze.map[i - 2][j - 1].isWay = 1; maze.map[x - 2][y - 1].isWay = 1; }
                else if (maxdp == dp[i - 1][j][x][y - 1]) { maze.map[i - 2][j - 1].isWay = 1; maze.map[x - 1][y - 2].isWay = 1; }
                else if (maxdp == dp[i][j - 1][x - 1][y]) { maze.map[i - 1][j - 2].isWay = 1; maze.map[x - 2][y - 1].isWay = 1; }
                else if (maxdp == dp[i][j - 1][x][y - 1]) { maze.map[i - 1][j - 2].isWay = 1; maze.map[x - 1][y - 2].isWay = 1; }
              }
              else
              {
                if (maxdp == dp[i - 1][j][x][y - 1]) { maze.map[i - 2][j - 1].isWay = 1; maze.map[x - 1][y - 2].isWay = 1; }
                else if (maxdp == dp[i][j - 1][x][y - 1]) { maze.map[x - 1][y - 2].isWay = 1; }
              }
            }
      maze.map[0][0].isWay = 1;
      maze.map[0][1].isWay = 1;
      maze.map[1][0].isWay = 1;
      maze.map[m - 2][n - 1].isWay = 1;
      maze.map[m - 1][n - 2].isWay = 1;
      maze.map[m - 1][n - 1].isWay = 1;
      cout << dp[m][n - 1][m - 1][n] + Data[m][n] + Data[1][1] << endl;
      for (int i = 0; i < m; ++i)
      {
        for (int j = 0; j < n; ++j)
        {
          maze.map[i][j].place.row = i;
          maze.map[i][j].place.col = j;
        }
      }
      Mat img = Mat::zeros(40 * maze.row, 40 * maze.col, CV_8UC3);
      //相当于创建一张黑色的图，每个像素的每个通道都为0,Scalar(0,0,0)
      drawMaze(img, maze, Data);
      stack<Grid*> t_solution1 = maze_solution(maze, img, 1, Data);
      stack<Grid*> solution1;
      while (!t_solution1.empty())
      {
        solution1.push(t_solution1.top());
        t_solution1.pop();
      }
      while (!solution1.empty())
      {
        cout << solution1.top()->place;
        solution1.pop();
        if (!solution1.empty())
          cout << "->";
      }
      cout << endl;
      cout << "-------------------- " << endl;
    }
    return 0;
  }
```

###### 附注：动态求解迷宫参考 https://blog.csdn.net/sunshine2285/article/details/89302129
