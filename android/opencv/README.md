# OpenCV Problems

## CvException - Java 层使用 OpenCV 出现崩溃

异常如下：

```java
Process: io.xxx.xxx, PID: 6992
    CvException [org.opencv.core.CvException: cv::Exception: OpenCV(3.4.10) /build/3_4_pack-android/opencv/modules/imgproc/src/approx.cpp:689: error: (-215:Assertion failed) npoints >= 0 && (depth == CV_32S || depth == CV_32F) in function 'void cv::approxPolyDP(cv::InputArray, cv::OutputArray, double, bool)'
    ]
        at org.opencv.imgproc.Imgproc.approxPolyDP_0(Native Method)
        at org.opencv.imgproc.Imgproc.approxPolyDP(Imgproc.java:4080)
        at ...
```

代码调用片段如下：

```java
ArrayList<MatOfPoint> contours = new ArrayList<>();
Mat hierarchy = new Mat();
final int retr = needInside ? Imgproc.RETR_LIST : Imgproc.RETR_EXTERNAL;
Imgproc.findContours(thresholdOutput, contours, hierarchy, retr, Imgproc.CHAIN_APPROX_SIMPLE, new Point(0, 0));      
...
ArrayList<MatOfPoint> contoursPoly = new ArrayList<>(contours.size());
for (int i = 0; i < contoursSize; i++) {
  contoursPoly.add(new MatOfPoint());
}

for (int i = 0; i < contoursSize; i++) {
  MatOfPoint2f curve = new MatOfPoint2f(contours.get(i));
  MatOfPoint2f approxCurve = new MatOfPoint2f();
  Imgproc.approxPolyDP(curve, approxCurve, 200, true);
           ^
          crash
  ...
}
```

异常原因：类型转换出现异常，如下代码是错误的转换：

```java
new MatOfPoint2f(contours.get(i));
```

解决方案：

先创建空的 `MatOfPoint2f` 对象，再使用 `convertTo` 方法进行转换。


```java
Imgproc.findContours(gray, contours, new Mat(), Imgproc.RETR_LIST, Imgproc.CHAIN_APPROX_SIMPLE);

for(int i=0;i<contours.size();i++){
  //Convert contours(i) from MatOfPoint to MatOfPoint2f
  contours.get(i).convertTo(mMOP2f1, CvType.CV_32FC2);
  //Processing on mMOP2f1 which is in type MatOfPoint2f
  Imgproc.approxPolyDP(mMOP2f1, mMOP2f2, approxDistance, true); 
  //Convert back to MatOfPoint and put the new values back into the contours list
  mMOP2f2.convertTo(contours.get(i), CvType.CV_32S);
}
```

引用：

[https://stackoverflow.com/questions/11273588/how-to-convert-matofpoint-to-matofpoint2f-in-opencv-java-api](https://stackoverflow.com/questions/11273588/how-to-convert-matofpoint-to-matofpoint2f-in-opencv-java-api)








