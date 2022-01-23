
###  Using the MVC Model :

In the MVC Model, We wrote the code for the actions , now we will write a set of basic functionality. we will start with the connections made for our application:

 we created three QstringList to store the rows in order to know the size, and we used QStandardItemModel that implements the QAbstractItemModel interface, which means that the model can be used to provide data in any view that supports that interface (such as QListView , QTableView and QTreeView).First We declared a three QstandardItemModel(one for Persistent, one for Pending and the third for Completed), besides we declared three QStringList for the same three tasks. In the Header file:


    private:
        Ui::toDoApp *ui;
        QStandardItemModel* m1;
        QStandardItemModel* m2;
        QStandardItemModel* m3;
    private:
        QStringList pesistent;
        QStringList Pending;
        QStringList Completed;
        
Then, we implement in the cpp file: In this implementation we have two parts, the First one for filling in the table using 'Task.append(line)' , the second one for the model view, we put the icon , the text and we set the model for the visibility (how to represent data and how to show data) :

    toDoApp::toDoApp(QWidget *parent)
        : QMainWindow(parent)
        , ui(new Ui::toDoApp)
    {
        ui->setupUi(this);
        setUpMainWidget();
        createActions();
        makeConnexions();
        createToolbars();
        createMenus();
        setWindowTitle("ToDoApp");
        m1 = new QStandardItemModel();
        m2 = new QStandardItemModel();
        m3 = new QStandardItemModel();
        QFile file("/Users/tuf/Downloads/save.txt");
        if (!file.open(QIODevice::ReadOnly | QIODevice::Text))
            return;
        while (!file.atEnd()) {
            QString line = file.readLine();
            QStandardItem *tmp = new QStandardItem();
            if(line.at(0)== "1"){
                pesistent.append(line);
                tmp->setIcon(QIcon(":/newtask_icon.png"));
                tmp->setText(line+" ");
                m1->appendRow(tmp);
                ui->pesistent->setModel(m1);
            }else if(line.at(0)== "2"){
                Pending.append(line);
                tmp->setIcon(QIcon(":/pending_icon.png"));
                tmp->setText(line+" ");
                m2->appendRow(tmp);
                ui->Pending->setModel(m2);
            }else if(line.at(0)== "3"){
                Completed.append(line);
                tmp->setIcon(QIcon(":/completed_icon.png"));
                tmp->setText(line+" ");
                m3->appendRow(tmp);
                ui->Completed->setModel(m3);
            }
        }
    
    }
In the ui Form using QDesigner , we used ListView with splitters :

![enter image description here](https://user-images.githubusercontent.com/93097467/150695766-b5bb5d06-8258-4777-9b06-80e6170fe87b.png)

ToDo ui Illustration

2.we add the function for the newTask action,we created a Dialog for the user to add tasks, for that, first we created a Form Class, we use the designer and we obtain the form of AddNew, in addition we added some methods to get the content of our line Edit, checkBox, comboBox and the Date Edit. Here is the Form of Add new task:

![enter image description here](https://user-images.githubusercontent.com/93097467/150695791-5e7313ed-c4e4-4091-969c-4e4250729107.png)




first we add those lines to the header file of the dialog class :

    namespace Ui {
    class Dialog1;
    }
    
    class Dialog1 : public QDialog
    {
        Q_OBJECT
    
    public:
        explicit Dialog1(QWidget *parent = nullptr);
        ~Dialog1();
        void taskDialog();
        bool isChecked();
        void setdate(int a,int m, int j);
        void task(QString t);
        void tag(QString a);
        void finished(bool f);
        QDate getDate();
        QString getText();
    
    private:
        Ui::Dialog1 *ui;
    };

and in the cpp file we implement our methods:

    Dialog1::Dialog1(QWidget *parent) :
        QDialog(parent),
        ui(new Ui::Dialog1)
    {
        ui->setupUi(this);
        QDate date =QDate::currentDate();
    
        ui->dateEdit->setMinimumDate(date);
    
        ui->dateEdit->setDate(date);
    }
    
    Dialog1::~Dialog1()
    {
        delete ui;
    }
    void Dialog1::setdate(int a, int m, int j){
    
        QDate d;
    
        d.setDate(a,m,j);
    
        ui->dateEdit->setDate(d);
    
    }
    void Dialog1::task(QString t){
        ui->lineEdit->setText(t);
    }
    void Dialog1::tag(QString a){
        ui->comboBox->setCurrentText(a);
    }
    
    void Dialog1::finished(bool f){
        ui->checkBox->setChecked(f);
    
    }
    bool Dialog1::isChecked(){
        if(ui->checkBox->isChecked()){
            return true;
        }
        return false;
    }
    QDate Dialog1::getDate(){
        return ui->dateEdit->date();
    }
    QString Dialog1::getText(){
        QString a= ui->lineEdit->text()+"Due :" + ui->dateEdit->text() + "Tag :" + ui->comboBox->currentText() + "";
        return a;
    }
    
    }
In addition,we have declared a private slot called Newtask:

    private slots:
        void NewTask();

We created a connexion between the newtask action and the method :

    connect(newtask,&QAction::triggered,this,&toDoApp::NewTask);
Then, we implement the function in the cpp file:

    void toDoApp::NewTask()
    {
    
        Dialog1 dialog;
        auto replu = dialog.exec();
        if(replu==Dialog1::Accepted){
            QString text = dialog.getText();
            QStandardItem *tmp = new QStandardItem;
            if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
                tmp->setIcon(QIcon(":/Todayicon.png"));
                tmp->setText("1 "+text+" ");
                m1->appendRow(tmp);
                ui->pesistent->setModel(m1);
                pesistent.append(text);
            }
            else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
    
                tmp->setIcon(QIcon(":/pending_icon.png"));
                tmp->setText("2 "+text+" ");
                m2->appendRow(tmp);
                ui->Pending->setModel(m2);
                Pending.append(text);
            }
            else if(dialog.isChecked()){
                tmp->setIcon(QIcon(":/completed_icon.png"));
                tmp->setText("3 "+text+" ");
    
                m3->appendRow(tmp);
    
                ui->Completed->setModel(m3);
                Completed.append(text);
    
            }
        }
    
    }
2.We added a closeEvent that save the data in a file int the format of txt First, we declare the slots in the header file:

    protected:
      void closeEvent(QCloseEvent *e) override;
      
Then we implement the function in the cpp file :

    void toDoApp::closeEvent(QCloseEvent *e){
    
      QFile file("/Users/tuf/Downloads/save.txt");
      if (file.open(QIODevice::ReadWrite| QIODevice::Text)){
    
          QTextStream out(&file);
          for (int i=0;i<pesistent.size() ;i++ ) {
              out<< "1"<< pesistent[i] << Qt::endl;
          }
          for (int i=0;i<Pending.size() ;i++ ) {
              out<< "2"<< Pending[i] << Qt::endl;
          }
          for (int i=0;i<Completed.size() ;i++ ) {
              out<< "3"<< Completed[i] << Qt::endl;
          }
          file.close();
      }
    }
    
3.For open the previous data , we added some line to open our file.txt:

    QFile file("/Users/tuf /Downloads/save.txt");
        if (!file.open(QIODevice::ReadOnly | QIODevice::Text))
            return;
        while (!file.atEnd()) {
            QString line = file.readLine();
            QStandardItem *tmp = new QStandardItem();
            if(line.at(0)== "1"){
                pesistent.append(line);
                tmp->setIcon(QIcon(":/newtask_icon.png"));
                tmp->setText(line+" ");
                m1->appendRow(tmp);
                ui->pesistent->setModel(m1);
            }else if(line.at(0)== "2"){
                Pending.append(line);
                tmp->setIcon(QIcon(":/pending_icon.png"));
                tmp->setText(line+" ");
                m2->appendRow(tmp);
                ui->Pending->setModel(m2);
            }else if(line.at(0)== "3"){
                Completed.append(line);
                tmp->setIcon(QIcon(":/completed_icon.png"));
                tmp->setText(line+" ");
                m3->appendRow(tmp);
                ui->Completed->setModel(m3);
            }
        }
    
    }
    
![enter image description here](https://user-images.githubusercontent.com/93097467/150695808-84446be0-1533-447c-be23-38089d54efac.jpg)


4-we added a slots called Pending slot and completed slot, it is used for the visibility of the task, we use it to show the task that we want to set it visible First, we declare the slots in the header file:

    private slots:
        void PendingSlot();
        void CompletedSlot();
Then we implement the function in the cpp file :

    void toDoApp::PendingSlot(){
        if(ui->Pending->isVisible()){
            ui->Pending->hide();
    }else{
       ui->Pending->show();
        }
    }
    
    void toDoApp::CompletedSlot(){
        if(ui->Completed->isVisible()){
            ui->Completed->hide();
    }else{
       ui->Completed->show();
        }
    }

We add the connexion of these slots to enable the actions to show or hide our listWidget:

    connect(pending,&QAction::triggered,this,&toDoApp::PendingSlot);
    connect(completed,&QAction::triggered,this,&toDoApp::CompletedSlot);
 
![enter image description here](https://user-images.githubusercontent.com/93097467/150695821-185f34da-2558-4d8a-a850-66a0fe19419c.png)

5- we added a slots called quit to close the window, and the about and aboutQt slots:

First, we declare the slots in the header file:

    private slots:
        void quit();
        void aboutslot();
        void aboutQtslot();
Then we implement the functions in the cpp file:

    void toDoApp::quit(){
        auto reply = QMessageBox::question(this, "Exit","Do you really want to quit?");
        if(reply == QMessageBox::Yes)
            qApp->exit();
    }
    void toDoApp::aboutslot(){
        QMessageBox::about(this,"about","to do app is an app to manage tasks");;
    }
    void toDoApp::aboutQtslot(){
        QMessageBox::aboutQt(this, "Your Qt");
    }

![enter image description here](https://user-images.githubusercontent.com/93097467/150689537-79e5a933-05a9-4389-b2f9-ddeb1cf7484f.png)
![enter image description here](https://user-images.githubusercontent.com/93097467/150689578-98b7d759-cb2e-44c8-b31a-7d06e9712c63.png)
![enter image description here](https://user-images.githubusercontent.com/93097467/150689592-12fb4f54-14d5-48d5-a87b-70b86fe5ccd6.png)
6- we added a slot called clear for delete the content of our list views : First, we declare the slots in the header file:

    protected:
        void closeEvent(QCloseEvent *e) override;

Then we implement the functions in the cpp file:

    void toDoApp::ClearSlot()
    {
    
     QFile file("/Users/tuf/Downloads/save.txt");
     if (file.open(QIODevice::ReadWrite| QIODevice::Text)){
         file.resize(0);
    
     }
    }

Finally here is the code of the header file:

  

      QT_BEGIN_NAMESPACE
        namespace Ui { class toDoApp; }
        QT_END_NAMESPACE
        
        class toDoApp : public QMainWindow
        {
          Q_OBJECT
        
        public:
          toDoApp(QWidget *parent = nullptr);
         ~toDoApp();
        private:
          Ui::toDoApp *ui;
          QStandardItemModel* m1;
          QStandardItemModel* m2;
          QStandardItemModel* m3;
        
        protected:
          void setUpMainWidget();
          void createActions();
          void makeConnexions();
          void createMenus();
          void createToolbars();
          void closeEvent(QCloseEvent *e) override;
          void dropEvent(QDropEvent *e) override;
        
        private slots:
          void quit();
          void NewTask();
          void aboutslot();
          void aboutQtslot();
          void ClearSlot();
        
          void PendingSlot();
          void CompletedSlot();
        private:
        
        
          QMenu *fileMenu;
          QMenu *optionsMenu;
          QMenu *HelpMenu;
          QMenu *toolsMenu;
        private:
          QAction *newtask;
          QAction *completed;
          QAction *pending;
          QAction *about;
          QAction *aboutQt;
          QAction *Close;
          QAction *Clear;
        
        private:
          QStringList pesistent;
          QStringList Pending;
          QStringList Completed;
        
        };
        
And here is the implemenation of all functions:

     toDoApp::toDoApp(QWidget *parent)
        : QMainWindow(parent)
        , ui(new Ui::toDoApp)
    {
        ui->setupUi(this);
        setUpMainWidget();
        createActions();
        makeConnexions();
        createToolbars();
        createMenus();
        setWindowTitle("ToDoApp");
    
        m1 = new QStandardItemModel();
        m2 = new QStandardItemModel();
        m3 = new QStandardItemModel();
    
        QFile file("/Users/tuf/Downloads/save.txt");
        if (!file.open(QIODevice::ReadOnly | QIODevice::Text))
            return;
    
        while (!file.atEnd()) {
            QString line = file.readLine();
            QStandardItem *tmp = new QStandardItem();
    
            if(line.at(0)== "1"){
    
                pesistent.append(line);
                tmp->setIcon(QIcon(":/newtask_icon.png"));
                tmp->setText(line+" ");
    
                m1->appendRow(tmp);
    
                ui->pesistent->setModel(m1);
            }else if(line.at(0)== "2"){
    
                Pending.append(line);
                tmp->setIcon(QIcon(":/pending_icon.png"));
                tmp->setText(line+" ");
    
                m2->appendRow(tmp);
    
                ui->Pending->setModel(m2);
            }else if(line.at(0)== "3"){
                Completed.append(line);
    
                tmp->setIcon(QIcon(":/completed_icon.png"));
                tmp->setText(line+" ");
    
                m3->appendRow(tmp);
    
                ui->Completed->setModel(m3);
            }
        }
    
    }
    
    toDoApp::~toDoApp()
    {
        delete ui;
        delete newtask;
        delete pending;
        delete completed;
        delete about;
        delete aboutQt;
        delete Close;
    
        delete fileMenu;
        delete HelpMenu;
        delete optionsMenu;
        delete toolsMenu;
        delete Clear;
    }
    void toDoApp::createActions()
    {
    
      QPixmap newIcon(":/newtask_icon.png");
      newtask= new QAction(newIcon,"&New",this);
      newtask->setShortcut(tr("CTRL+N"));
    
      QPixmap pendingIcon(":/pending_icon.png");
      pending= new QAction(pendingIcon,"&Pending",this);
    
      QPixmap completedIcon(":/completed_icon.png");
    
      completed= new QAction(completedIcon,"&Completed",this);
    
      QPixmap aboutIcon(":/about_icon.png");
      about= new QAction(aboutIcon,"&About",this);
      QPixmap aboutQtIcon(":/aboutQt_icon.png");
      aboutQt= new QAction(aboutQtIcon,"&AboutQt",this);
    
      QPixmap closeIcon(":/close_icon.png");
      Close= new QAction(closeIcon,"&Close",this);
      Close->setShortcut(tr("F5"));
      QPixmap clearIcon(":/clearicon.png");
      Clear= new QAction(clearIcon,"&Clear",this);
      Clear->setShortcut(tr("F7"));
    
    }
    
    void toDoApp::createMenus()
    {
        fileMenu = menuBar()->addMenu("&File");
        fileMenu->addAction(newtask);
        fileMenu->addAction(Close);
    
        optionsMenu = menuBar()->addMenu("&Options");
        optionsMenu->addAction(Clear);
        optionsMenu->addAction(completed);
        optionsMenu->addAction(pending);
    
        HelpMenu= menuBar()->addMenu("&Help");
        HelpMenu->addAction(about);
        HelpMenu->addAction(aboutQt);
    }
    void toDoApp::createToolbars()
    {
    
            auto toolbar1 = addToolBar("File");
            toolbar1->addAction(newtask);
            toolbar1->addAction(Close);
            auto toolbar2 = addToolBar("Options");
            toolbar2->addAction(Clear);
            toolbar2->addAction(pending);
            toolbar2->addAction(completed);
    }
    void toDoApp::makeConnexions()
    {
        connect(Close,&QAction::triggered,this,&toDoApp::quit);
        connect(Clear,&QAction::triggered,this,&toDoApp::ClearSlot);
        connect(newtask,&QAction::triggered,this,&toDoApp::NewTask);
        connect(about,&QAction::triggered,this,&toDoApp::aboutslot);
        connect(aboutQt,&QAction::triggered,this,&toDoApp::aboutQtslot);
        connect(pending,&QAction::triggered,this,&toDoApp::PendingSlot);
        connect(completed,&QAction::triggered,this,&toDoApp::CompletedSlot);
    
    }
    void toDoApp::setUpMainWidget()
    {
    
        ui->Pending->setVisible(false);
        ui->Completed->setVisible(false);
        ui->Pending->setDragDropMode(QAbstractItemView::DragDrop);
        ui->pesistent->setDragDropMode(QAbstractItemView::DragDrop);
        ui->Completed->setDragDropMode(QAbstractItemView::DragDrop);
        ui->Pending->setDefaultDropAction(Qt::MoveAction);
        ui->Completed->setDefaultDropAction(Qt::MoveAction);
        ui->pesistent->setDefaultDropAction(Qt::MoveAction);
    
    }
    
    void toDoApp::quit(){
        auto reply = QMessageBox::question(this, "Exit","Do you really want to quit?");
        if(reply == QMessageBox::Yes)
            qApp->exit();
    }
    void toDoApp::NewTask()
    {
    
        Dialog1 dialog;
        auto replu = dialog.exec();
        if(replu==Dialog1::Accepted){
            QString text = dialog.getText();
            QStandardItem *tmp = new QStandardItem;
    
            if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
                tmp->setIcon(QIcon(":/Todayicon.png"));
                tmp->setText("1 "+text+" ");
    
                m1->appendRow(tmp);
    
                ui->pesistent->setModel(m1);
                pesistent.append(text);
    
            }
            else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
    
                tmp->setIcon(QIcon(":/pending_icon.png"));
                tmp->setText("2 "+text+" ");
    
                m2->appendRow(tmp);
    
                ui->Pending->setModel(m2);
                Pending.append(text);
    
            }
            else if(dialog.isChecked()){
    
                tmp->setIcon(QIcon(":/completed_icon.png"));
                tmp->setText("3 "+text+" ");
    
                m3->appendRow(tmp);
    
                ui->Completed->setModel(m3);
                Completed.append(text);
    
            }
        }
    
    }
    void toDoApp::aboutslot()
    {
        QMessageBox::about(this,"about","to do app is an app to manage tasks");
    }
    void toDoApp::aboutQtslot()
    {
        QMessageBox::aboutQt(this,"Your Qt");
    }
    
    void toDoApp::PendingSlot(){
        if(ui->Pending->isVisible()){
            ui->Pending->hide();
    }else{
       ui->Pending->show();
        }
    }
    
    void toDoApp::CompletedSlot(){
        if(ui->Completed->isVisible()){
            ui->Completed->hide();
    }else{
       ui->Completed->show();
        }
    }
    
    void toDoApp::closeEvent(QCloseEvent *e){
    
        QFile file("/Users/tuf/Downloads/save.txt");
        if (file.open(QIODevice::ReadWrite| QIODevice::Text)){
    
            QTextStream out(&file);
            for (int i=0;i<pesistent.size() ;i++ ) {
                out<< "1"<< pesistent[i] << Qt::endl;
            }
            for (int i=0;i<Pending.size() ;i++ ) {
                out<< "2"<< Pending[i] << Qt::endl;
            }
            for (int i=0;i<Completed.size() ;i++ ) {
                out<< "3"<< Completed[i] << Qt::endl;
            }
            file.close();
        }
    }
    
    void toDoApp::dropEvent(QDropEvent *e){
        Dialog1 dialog;
        auto reply= dialog.exec();
    }
    
    void toDoApp::ClearSlot()
    {
    
        QFile file("/Users/tuf/Downloads/save.txt");
        if (file.open(QIODevice::ReadWrite| QIODevice::Text)){
            file.resize(0);
    
        }
    }

