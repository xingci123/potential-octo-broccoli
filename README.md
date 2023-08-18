#include "studentdlg.h"
#include "ui_studentdlg.h"
#include <QApplication>
#include <QTableView>
#include <QSqlDatabase>
#include <QSqlQueryModel>
#include<QSqlTableModel>
#include <QHeaderView>
#include <QAbstractItemView>
#include <QStyledItemDelegate>
#include <QTextOption>
#include<QString>


StudentDlg::StudentDlg(QWidget *parent)
    : QDialog(parent)
    , ui(new Ui::StudentDlg)
{
    ui->setupUi(this);

    // 调用函数创建且打开数据库
    CreateDatabaseFunc();

    // 调用函数创建数据表
    CreateTableFunc();


    //加载数据到tableView_zhaosheng控件上
    LoadDataToTableView();

}
StudentDlg::~StudentDlg()
{
    delete ui;
}
void StudentDlg::CreateDatabaseFunc()  // 创建SQLite数据库
{
    // 1：添加数据库驱动
    sqldb=QSqlDatabase::addDatabase("QSQLITE");

    // 2：设置数据库名称
    sqldb.setDatabaseName("studentmis.db");

    // 3：打开此数据库是否成功
    if(sqldb.open()==true)
    {
        QMessageBox::information(0,"正确","恭喜你，数据库打开成功！",QMessageBox::Ok);
    }
    else
    {
        QMessageBox::critical(0,"错误","数据库打开失败，请重新检测！",QMessageBox::Ok);
    }
}


void StudentDlg::CreateTableFunc()
{
    QSqlQuery createquery;
    //创建sql语句
    QString strsql=QString("create table student("
                             "name varchar(50)  not null,"
                             "baomingqingkuang varchar(50) not null,"

                             "score decimal(10,2) not null,"
                             "jiaofeiqingkuang varchar(50)  not null,"
                             "phone varchar(20)  not null,"
                             "beizhu char(10)  )");

    // 执行SQL语句
    if(createquery.exec(strsql)==false)
    {
        QMessageBox::critical(0,"失败","数据表创建失败，请重新检查！",QMessageBox::Ok);
    }
    else
    {
        QMessageBox::information(0,"成功","恭喜你，数据表创建成功！",QMessageBox::Ok);
    }

}//创建SQLite数据表


//加载数据到tableView函数
void StudentDlg::LoadDataToTableView()
{
    // 创建自定义的查询模型
    QSqlQueryModel *model = new QSqlQueryModel(this);
    model->setQuery("SELECT * FROM student");

    // 设置模型到tableView_zhaosheng控件
    ui->tableView_zhaosheng->setModel(model);

    // 设置表头自适应宽度
    QHeaderView *header = ui->tableView_zhaosheng->horizontalHeader();
    header->setSectionResizeMode(QHeaderView::Stretch);

    // 隐藏水平表头
    ui->tableView_zhaosheng->horizontalHeader()->hide();

}


void StudentDlg::QueryTableFunc()
{
}//执行查询操作
void StudentDlg::on_pushButtonSort_clicked()
{

}


void StudentDlg::on_pushButton_INSERT_clicked()
{
    QSqlQuery sqlquery;


    //姓名
    QString name =ui->lineEdit_NAME->text();
    if(name=="")
    {
        QMessageBox::critical(this,"失败","  提示：输入错误？学生姓名不能为空  ",QMessageBox::Ok);
        return ;
    }


    //报名情况
    QString baomingqingkuang =ui->lineEdit_BAOMINGQINGKUANG->text();
    if(baomingqingkuang=="")
    {
        QMessageBox::critical(this,"失败","  提示：输入错误？招生情况不能为空  ",QMessageBox::Ok);
        return ;
    }


    //分数
    double score =ui->lineEdit_SCORE->text().toDouble();
    if(score<100||score>750)
    {
        QMessageBox::critical(this,"失败","  提示：分数不符合阈值（100-750）请重新输入  ",QMessageBox::Ok);
        return ;
    }


    //缴费情况
    QString jiaofeiqingkuang =ui->lineEdit_JIAOFEIQINGKUANG->text();
    if(jiaofeiqingkuang=="")
    {
        QMessageBox::critical(this,"失败","  提示：输入错误？请填写学生缴费情况  ",QMessageBox::Ok);
        return ;
    }


    //联系方式
    QString phone =ui->lineEdit_PHONE->text();
    if(phone=="")
    {
        QMessageBox::critical(this,"失败","  提示：请填写学生有效联系方式  ",QMessageBox::Ok);
        return ;
    }

    //备注
    QString beizhu =ui->lineEdit_BEIZHU->text();


    //插入
    QString strs = QString("INSERT INTO student VALUES ('%1', '%2', %3, '%4', '%5', '%6')")
                       .arg(name).arg(baomingqingkuang).arg(score).arg(jiaofeiqingkuang)
                       .arg(phone).arg(beizhu);

    //实时刷新数据显示
    LoadDataToTableView();


    if (sqlquery.exec(strs) == false) {
        QMessageBox::critical(0, "失败", "数据录入失败，请重新检查！", QMessageBox::Ok);
        //实时刷新数据显示
        LoadDataToTableView();
    } else {
        QMessageBox::information(0, "成功", "恭喜你，学生数据录入成功！", QMessageBox::Ok);
        //实时刷新数据显示
        LoadDataToTableView();
    }
}

void StudentDlg::on_pushButton_DELETE_clicked()
{
    // 获取当前选中的行
    QModelIndexList selectedRows = ui->tableView_zhaosheng->selectionModel()->selectedRows();
    int count = selectedRows.size();
    if (count == 0)
    {
        QMessageBox::critical(this, "失败", "请选择要删除的学生信息（一行或多行）", QMessageBox::Ok);
        return;
    }

    // 提示用户是否确认删除
    QMessageBox::StandardButton reply = QMessageBox::question(this, "确认删除", QString("确定要删除选中的 %1 条学生信息吗？").arg(count),
                                                              QMessageBox::Yes | QMessageBox::No);
    if (reply != QMessageBox::Yes)
    {
        return;
    }

    // 执行删除操作
    QSqlQuery sqlQuery;
    bool success = true;

    foreach (const QModelIndex& index, selectedRows)
    {
        QString name = ui->tableView_zhaosheng->model()->index(index.row(), 0).data().toString();
        QString queryStr = QString("DELETE FROM student WHERE name = '%1'").arg(name);

        if (!sqlQuery.exec(queryStr))
        {
            success = false;
            qDebug() << "删除学生信息失败：" << sqlQuery.lastError().text();
        }
    }

    if (success)
    {
        QMessageBox::information(this, "成功", QString("成功删除 %1 条学生信息！").arg(count), QMessageBox::Ok);
    }
    else
    {
        QMessageBox::critical(this, "失败", "删除学生信息失败，请重试！", QMessageBox::Ok);
    }

    // 实时刷新数据显示
    LoadDataToTableView();
}
// 更改学生数据信息
void StudentDlg::on_pushButton_UPDATE_clicked()
{

}
void StudentDlg::on_pushButton_SEARCH_clicked()
{

}

void StudentDlg::on_pushButtonjioafeiqingkuang_clicked()
{

}

void StudentDlg::on_pushButtonphone_clicked()
{

}

void StudentDlg::on_pushButtonbeizhu_clicked()
{

}

