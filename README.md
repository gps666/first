#check.pro


#-------------------------------------------------
#
# Project created by QtCreator 2018-08-09T10:57:17
#
#-------------------------------------------------
#选择一个图片文件，获取到文件的路径名，QImage类加载文件，生成image对象，使用QLabel类的setPixmap函数将图片显示在界面上，QLabel类修。
#在打开文件同时，获取到当前目录的文件列表，保存到文件信息链表中，当需要对打开的文件上、下切换时，只需对当前链表的下表进行轮询
#图片的旋转使用QImage类的transformed行为来实现，使用成员变量保存下当前图片的旋转角度，每次触发旋转操作，将旋转角度以90度间隔增减
#图片的放大、缩小操作通过QImage类的scaled行为实现。重置image的尺寸，返回新的image对象

QT       += core gui

greaterThan(QT_MAJOR_VERSION, 4): QT += widgets

TARGET = check
TEMPLATE = app

DEFINES += QT_DEPRECATED_WARNINGS




SOURCES += main.cpp\
        widget.cpp

HEADERS  += widget.h

FORMS    += widget.ui


//widget.h

#ifndef WIDGET_H
#define WIDGET_H//宏定义

#include <QWidget>//头文件
#include<QLabel>
#include<QPushButton>
#include<QDebug>
#include<QMouseEvent>
#include<QWheelEvent>
#include<QKeyEvent>

#include<QScrollArea>
#include<QImage>
#include<QFileInfoList>
namespace Ui {
class Widget;
}

class Widget : public QWidget
{
    Q_OBJECT

public:

    Widget(QWidget *parent = 0);
    ~Widget();
public:
    QImage image;
    QString filename;
    QPixmap pix;
    int index;
    int imageAngle;
    QString path;
    QFileInfoList imgInfoList;

public:
    void imageShow();
    void getImInfoList();
public slots://各个函数
    void W_open();//打开图片所在文件
    void W_last();//上一个图
    void W_next();//下一个图
    void W_more();//放大
    void W_less();//缩小
    void W_left();//左旋
    void W_right();//右旋

private:
    Ui::Widget *ui;
    QLabel *imageLabel;
    QPushButton *previous;//各个按钮信号
    QPushButton *open;
    QPushButton *next;
    QPushButton *more;
    QPushButton *less;
    QPushButton *left;
    QPushButton *right;
    QPoint offset;//坐标点
protected:
    void mouseMoveEvent(QMouseEvent *event);//各个鼠标事件
    void mouseReleaseEvent(QMouseEvent *event);
    void mouseDoubleClickEvent(QMouseEvent * event);
};


#endif // WIDGET_H

//main.cpp


#include "widget.h"
#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    Widget w;//定义
    w.show();//显示

    return a.exec();
}

//widget.cpp


#include "widget.h"
#include "ui_widget.h"

#include<QVBoxLayout>
#include<QHBoxLayout>
#include<QString>
#include<QFileDialog>
#include<QMouseEvent>//添加头文件
#include<QLabel>
#include<QPixmap>
#include<QPainter>
#include<QScrollArea>
#include<QImage>
#include<QMessageBox>
#include<QString>
#include<QFileInfoList>
Widget::Widget(QWidget *parent)
    : QWidget(parent),
  ui(new Ui::Widget)
{
    ui->setupUi(this);

    setFixedSize(600,500);//设置长宽
    imageLabel = new QLabel(this);//申请相应内存
    more = new QPushButton("放大",this);
    previous = new QPushButton("上一张",this);
    open =new QPushButton("打开图片",this);
    next = new QPushButton("下一张",this);
    less = new QPushButton("缩小",this);
    left = new QPushButton("左转",this);
    right = new QPushButton("右转",this);
    //设置按钮的图标和名字
    QHBoxLayout *hbox = new QHBoxLayout;
    hbox->addWidget(previous);
    hbox->addWidget(open);
    hbox->addWidget(next);
    hbox->addWidget(more);
    hbox->addWidget(less);
    hbox->addWidget(left);
    hbox->addWidget(right);
    QVBoxLayout *vbox = new QVBoxLayout;
    vbox->addWidget(imageLabel);
    vbox->addLayout(hbox);
    setLayout(vbox);
    connect(open,SIGNAL(clicked()),this,SLOT(W_open()));//作槽函数触发
    connect(previous,SIGNAL(clicked()),this,SLOT(W_last()));
    connect(next,SIGNAL(clicked()),this,SLOT(W_next()));
    connect(more,SIGNAL(clicked()),this,SLOT(W_more()));
    connect(less,SIGNAL(clicked()),this,SLOT(W_less()));
    connect(left,SIGNAL(clicked()),this,SLOT(W_left()));
    connect(right,SIGNAL(clicked()),this,SLOT(W_right()));
    QCursor cursor;//创建光标对象
    cursor.setShape(Qt::OpenHandCursor);//设置光标形状
    setCursor(cursor);//使用光标
    setMouseTracking(true);//设置光标跟踪
}

void Widget::W_open()//打开图片
{
         filename = QFileDialog::getOpenFileName(this, tr("Select image:"),".", tr("Images (*.png *.bmp *.jpg *.gif)"));
        if (filename.isEmpty())
            return;
        if (!image.load(filename)) {
            QMessageBox::information(this, tr("Error"), tr("Open file error"));
            return;
        }
        getImInfoList();
        imageShow();

}
void Widget::imageShow()//显示函数
{

    QPixmap pixmap = QPixmap::fromImage(image);
    pix = pixmap;//全局pix
    QSize imagesize = pixmap.size();
    imageLabel->resize(imagesize);//重新调整label大小以适应图片大小
    imageLabel->setPixmap(pixmap);
}
void Widget::getImInfoList()//获取图片列表
{
    imgInfoList.clear();
    path = QFileInfo(filename).absolutePath();//绝对路径
    QDir dir = QFileInfo(filename).absoluteDir();
    QFileInfoList infoList = dir.entryInfoList(QDir::Files);
    QFileInfo info;
    for (int i = 0; i < infoList.count(); i++)
    {
        info = infoList.at(i);
        QString suffix = info.suffix();
        if (suffix == "jpg" || suffix == "bmp" || suffix == "png") {
            imgInfoList.append(info);
        }
    }

    QFileInfo curImageInfo = QFileInfo(filename);
    for (int j = 0; j < imgInfoList.count(); j++) {

        info = imgInfoList.at(j);
        if (info.fileName() == curImageInfo.fileName()) {
            index = j;
    }
    }
}


void Widget::W_next()//下一张
{
    index = index + 1;
        int count = imgInfoList.count();
        if (index == count) {
            index = 0;
     }
        filename.clear();
        filename.append(path);
        filename += "/";
        filename += imgInfoList.at(index).fileName();
        if (!image.load(filename)) {
            QMessageBox::information(this,tr("Error"),tr("Open file Error"));
            return;
        }
        imageShow();

}
void Widget::W_left()//向左转
{
    QMatrix matrix;
        QPixmap pixmap;
        imageAngle += 1;
        imageAngle = imageAngle % 4;
        matrix.rotate(imageAngle*90);//旋转90度
        image.load(filename);
        image = image.transformed(matrix);
        imageShow();
}
void Widget::W_right()//向右转
{
    QMatrix matrix;
        QPixmap pixmap;
        imageAngle += 3;
        imageAngle = imageAngle % 4;
        matrix.rotate(imageAngle * 90);//90°
        image.load(filename);
        image = image.transformed(matrix);
        imageShow();
}

void Widget::W_last()//上一张
{
    index = index - 1;
        int count = imgInfoList.count();
        if (index == -1)
            index = count - 1;
        filename.clear();
        filename.append(path);
        filename += "/";
        filename += imgInfoList.at(index).fileName();

        if (!image.load(filename)) {
            QMessageBox::information(this,tr("Error"),tr("Open file error"));
            return;
        }
        imageShow();

}

void Widget::W_more()//放大
{
    QMatrix matrix;
        QPixmap pixmap;
        image.load(filename);
        matrix.rotate(imageAngle * 90);
        image = image.transformed(matrix);
        pixmap = QPixmap::fromImage(image);
        pixmap = pixmap.scaled(pix.width()*1.2, pix.height()*1.2, Qt::KeepAspectRatio);
        pix = pix.scaled(pix.width()*1.2, pix.height()*1.2, Qt::KeepAspectRatio);

        imageLabel->setPixmap(pixmap);
}
void Widget::W_less()//缩小
{
    QMatrix matrix;
         QPixmap pixmap;
         image.load(filename);
         matrix.rotate(imageAngle * 90);
         image = image.transformed(matrix);
         pixmap = QPixmap::fromImage(image);
         pixmap = pixmap.scaled(pix.width()*0.8, pix.height()*0.8, Qt::KeepAspectRatio);
         pix = pix.scaled(pix.width()*0.8, pix.height()*0.8, Qt::KeepAspectRatio);

        imageLabel->setPixmap(pixmap);
}
void Widget::mouseDoubleClickEvent(QMouseEvent *event)//鼠标双击事件
{
    if(event->button()==Qt::LeftButton){
        if(windowState()!=Qt::WindowFullScreen)
            setWindowState(Qt::WindowFullScreen);
        else setWindowState(Qt::WindowNoState);
    }
}

void Widget::mouseMoveEvent(QMouseEvent *event)//鼠标移动事件
{
    if(event->buttons() & Qt::LeftButton){
        QPoint temp;
        temp=event->globalPos() - offset;
        move(temp);
    }
}
void Widget::mouseReleaseEvent(QMouseEvent *event)//取消事件
{
    Q_UNUSED(event);
    QApplication::restoreOverrideCursor();
}

Widget::~Widget(){
   delete ui;
}//析构函数
