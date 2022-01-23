# doList-Model-Based
# doList.cpp

#include "doList.h"
#include "ui_todoapp.h"
#include <QMessageBox>
#include <task.h>

todoApp::todoApp(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::todoApp)
{
    ui->setupUi(this);



    todayTaskModel = new QSqlQueryModel;
    pendingTaskModel = new QSqlQueryModel;
    finishedTaskModel = new QSqlQueryModel;


    connectDB();


    showDB();


    connect(ui->todayTask, &QTableView::doubleClicked, this, &todoApp::editTaskSlot);

}

todoApp::~todoApp()
{
    delete ui;
    delete todayTaskModel;
    delete pendingTaskModel;
    delete finishedTaskModel;
}

void todoApp::connectDB()
{

    db = QSqlDatabase::addDatabase("QSQLITE");


    db.setDatabaseName(this->filename);


    if(!db.open())
        QMessageBox::critical(this, "FAILED", "DB is not opened");


    auto todayTaskQuery = QSqlQuery(db);


    QString createTodayTask{"CREATE TABLE IF NOT EXISTS todaytasks"
                            "(id INT AUTO_INCREMENT,description VARCHAR(80),"
                            "date DATE, tag VARCHAR(10), finished BOOLEAN)"};


    if(!todayTaskQuery.exec(createTodayTask))
        QMessageBox::critical(this,"Info","Cannot create the table");


    auto  pendingTaskQuery = QSqlQuery(db);
    QString createPendingTask{"CREATE TABLE IF NOT EXISTS pendingtasks"
                              "(description VARCHAR(80),date DATE,"
                              "tag VARCHAR(10),finished BOOLEAN)"};
    if(!pendingTaskQuery.exec(createPendingTask))
        QMessageBox::critical(this,"Info","Cannot create the table");


    auto  finishedTaskQuery = QSqlQuery(db);
    QString createfinishedTask{"CREATE TABLE IF NOT EXISTS finishedtasks"
                               "(description VARCHAR(80),date DATE,"
                               "tag VARCHAR(10),finished BOOLEAN)"};
    if(!finishedTaskQuery.exec(createfinishedTask))
        QMessageBox::critical(this,"Info","Cannot create the table");
}

void todoApp::showDB()
{

        auto todayQuery = QSqlQuery(db);
        QString today{"SELECT * FROM todaytasks"};
        todayQuery.exec(today);
        todayTaskModel->setQuery(todayQuery);
        ui->todayTask->setModel(todayTaskModel);


         auto pendingQuery = QSqlQuery(db);
         QString pending{"SELECT * FROM pendingtasks"};
         pendingQuery.exec(pending);
         pendingTaskModel->setQuery(pendingQuery);
         ui->pendingTask->setModel(pendingTaskModel);



        auto finishedQuery = QSqlQuery(db);
        QString finished{"SELECT * FROM finishedtasks"};
        finishedQuery.exec(finished);
        finishedTaskModel->setQuery(finishedQuery);
        ui->finishedTask->setModel(finishedTaskModel);

}

void todoApp::on_actionNew_Task_triggered()
{

    Task T;


    auto reply = T.exec();


    if(reply == Task::Accepted)
    {
        addEnty(T.getDesc(), T.getDate(), T.getTag(), T.getStatus());

        if(T.getDate() == QDate::currentDate())
        {

            auto  query = QSqlQuery(db);
            QString view{"SELECT * FROM todaytasks"};
            query.exec(view);
            todayTaskModel->setQuery(query);
            ui->todayTask->setModel(todayTaskModel);
        }
        else if(T.getDate() > QDate::currentDate())
        {

            auto  query = QSqlQuery(db);
            QString view{"SELECT * FROM pendingtasks"};
            query.exec(view);
            pendingTaskModel->setQuery(query);
            ui->pendingTask->setModel(pendingTaskModel);
        }
        else
        {

            auto  query = QSqlQuery(db);
            QString view{"SELECT * FROM finishedtasks"};
            query.exec(view);
            finishedTaskModel->setQuery(query);
            ui->finishedTask->setModel(finishedTaskModel);
        }

    }

}

void todoApp::addEnty(QString desc, QDate date, QString tag, QString finished)
{

    if(date == QDate::currentDate())
    {

        auto query = QSqlQuery(db);


        QString addInfo{"INSERT INTO todaytasks (description, date, tag, finished)"
                        " VALUES('%1','%2', '%3', '%4')"};


        if(!query.exec(addInfo.arg(desc).arg(date.toString()).arg(tag).arg(finished)))
            QMessageBox::critical(this,"Info","Cannot add the Entry");


        todayTaskModel->setQuery(query);
    }
    else if(date > QDate::currentDate())
    {
        auto query = QSqlQuery(db);
        QString addInfo{"INSERT INTO pendingtasks VALUES('%1','%2', '%3', '%4')"};
        if(!query.exec(addInfo.arg(desc).arg(date.toString()).arg(tag).arg(finished)))
            QMessageBox::critical(this,"Info","Cannot add the Entry");
        pendingTaskModel->setQuery(query);
    }
    else
    {
        auto query = QSqlQuery(db);
        QString addInfo{"INSERT INTO finishedtasks VALUES('%1','%2', '%3', '%4')"};
        if(!query.exec(addInfo.arg(desc).arg(date.toString()).arg(tag).arg(finished)))
            QMessageBox::critical(this,"Info","Cannot add the Entry");
        finishedTaskModel->setQuery(query);
    }
}

void todoApp::on_action_Exit_triggered()
{
    QApplication::quit();
}

void todoApp::on_actionToday_Task_triggered()
{
    if(ui->todayTask->isVisible() && ui->todayLabel->isVisible())
    {
        ui->todayTask->setVisible(false);
        ui->todayLabel->setVisible(false);
    }
    else
    {
        ui->todayTask->setVisible(true);
        ui->todayLabel->setVisible(true);
    }
}

void todoApp::on_actionPending_Task_triggered()
{
    if(ui->pendingTask->isVisible() && ui->pendingLabel->isVisible())
    {
        ui->pendingTask->setVisible(false);
        ui->pendingLabel->setVisible(false);
    }
    else
    {
        ui->pendingTask->setVisible(true);
        ui->pendingLabel->setVisible(true);
    }
}

void todoApp::on_actionFinished_Tasks_triggered()
{
    if(ui->finishedTask->isVisible() && ui->finishedLabel->isVisible())
    {
        ui->finishedTask->setVisible(false);
        ui->finishedLabel->setVisible(false);
    }
    else
    {
        ui->finishedTask->setVisible(true);
        ui->finishedLabel->setVisible(true);
    }
}

void todoApp::on_actionDelete_Tasks_triggered()
{

    QModelIndexList selectToday = ui->todayTask->selectionModel()->selectedRows();


    for(int i=0; i<selectToday.count(); i++)
    {

        auto query = QSqlQuery(db);


        QString del{"DELETE FROM todaytasks WHERE description="
                    "'"+selectToday[i].data(Qt::EditRole).toString()+"'"};


        if(!query.exec(del))
            qDebug() << "Cannot delete row!";


        todayTaskModel->setQuery(query);


        todayTaskModel->submit();


        auto todayQuery = QSqlQuery(db);
        QString today{"SELECT * FROM todaytasks"};
        todayQuery.exec(today);
        todayTaskModel->setQuery(todayQuery);
        ui->todayTask->setModel(todayTaskModel);

    }


    QModelIndexList selectPending = ui->pendingTask->selectionModel()->selectedRows();


    for(int i=0; i<selectPending.count(); i++)
    {

        auto query = QSqlQuery(db);


        QString del{"DELETE FROM pendingtasks WHERE description="
                    "'"+selectPending[i].data(Qt::EditRole).toString()+"'"};


        if(!query.exec(del))
            qDebug() << "Cannot delete row!";


        pendingTaskModel->setQuery(query);


        pendingTaskModel->submit();


        auto pendingQuery = QSqlQuery(db);
        QString pending{"SELECT * FROM pendingtasks"};
        pendingQuery.exec(pending);
        pendingTaskModel->setQuery(pendingQuery);
        ui->pendingTask->setModel(pendingTaskModel);

    }


    QModelIndexList selectFinished = ui->finishedTask->selectionModel()->selectedRows();


    for(int i=0; i<selectFinished.count(); i++)
    {

        auto query = QSqlQuery(db);


        QString del{"DELETE FROM finishedtasks WHERE description="
                    "'"+selectFinished[i].data(Qt::EditRole).toString()+"'"};


        if(!query.exec(del))
            qDebug() << "Cannot delete row!";


        finishedTaskModel->setQuery(query);


        finishedTaskModel->submit();


        auto finishedQuery = QSqlQuery(db);
        QString finished{"SELECT * FROM finishedtasks"};
        finishedQuery.exec(finished);
        finishedTaskModel->setQuery(finishedQuery);
        ui->finishedTask->setModel(finishedTaskModel);

    }
}

void todoApp::editTaskSlot()
{
    auto currRow = ui->todayTask->currentIndex().row();

    qDebug() << currRow;

    Task T;
    auto reply = T.exec();

    if(reply == Task::Accepted)
    {
        auto query = QSqlQuery(db);


        QString addInfo{"UPDATE todaytasks (description, date, tag, finished)"
                        " VALUES('%1','%2', '%3', '%4') WHERE id = "};


        todayTaskModel->setQuery(query);
    }

}


# doList.h

#ifndef DOLIST_H
#define DOLIST_H

#include <QMainWindow>
#include <QtSql>
#include <QSqlQuery>
#include <QDir>

QT_BEGIN_NAMESPACE
namespace Ui { class todoApp; }
QT_END_NAMESPACE

class todoApp : public QMainWindow
{
    Q_OBJECT

public:
    todoApp(QWidget *parent = nullptr);
    ~todoApp();

    void connectDB();
    void addEnty(QString, QDate, QString, QString);
    void showDB();

private slots:
    void on_actionNew_Task_triggered();

    void on_actionDelete_Tasks_triggered();

    void on_action_Exit_triggered();

    void on_actionToday_Task_triggered();

    void on_actionPending_Task_triggered();

    void on_actionFinished_Tasks_triggered();

    void editTaskSlot();

private:
    Ui::todoApp *ui;
    QSqlDatabase db;
    QSqlQueryModel *todayTaskModel;
    QSqlQueryModel *pendingTaskModel;
    QSqlQueryModel *finishedTaskModel;
    QString filename = QDir::homePath()+ "/tasks.sqlite";;
};
#endif // DOLIST_H

# task.cpp

#include "task.h"
#include "ui_task.h"

Task::Task(QWidget *parent) :
    QDialog(parent),
    ui(new Ui::Task)
{
    ui->setupUi(this);


    this->setWindowTitle("New Task");

    ui->dateEdit->setDate(QDate::currentDate());
}

Task::~Task()
{
    delete ui;
}

QString Task::getDesc(){
    return "Description: " + ui->lineEdit->text();
}

QString Task::getStatus(){

    if(ui->finished->isChecked())
        return "Finished: Yes";
    else
        return "Finished: No";

}

QString Task::getTag(){
    return "Tag: " + ui->Tag->currentText();
}

QDate Task::getDate(){
    return ui->dateEdit->date();
}


void Task::setDesc(QString desc){
    ui->description->setText(desc);
}

void Task::setTag(QString currText){
    ui->Tag->setCurrentText(currText);
}

void Task::setDate(QString str){


    auto dateList = str.split("/");


    if(!dateList.isEmpty())
    {
        auto date = QDate(dateList.at(2).toInt(), dateList.at(1).toInt(), dateList.at(0).toInt());
        ui->dateEdit->setDate(date);
    }
}

void Task::setStatus(QString status){
    if(status == "Yes")
        ui->finished->setChecked(true);
    else
        ui->finished->setChecked(false);
}

# task.h

#ifndef TASK_H
#define TASK_H

#include <QDialog>

namespace Ui {
class Task;
}

class Task : public QDialog
{
    Q_OBJECT

public:
    explicit Task(QWidget *parent = nullptr);
    ~Task();

    QString getDesc();
    QString getStatus();
    QString getTag();
    QDate getDate();


    void setDesc(QString);
    void setStatus(QString);
    void setTag(QString);
    void setDate(QString date);

private:
    Ui::Task *ui;
};

#endif // TASK_H

main.cpp

#include "doList.h"

#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    todoApp w;
    w.show();
    return a.exec();
}
