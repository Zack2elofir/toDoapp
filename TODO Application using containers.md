## TODO Application using containers :

> The goal is to create a versatile ToDo app using (Item/Model) based containers.
**Widgets** are the primary elements for creating user interfaces in Qt. Widgets can display data and status information, receive user input, and provide a container for other widgets that should be grouped together. A widget that is not embedded in a parent widget is called a window.
A **Container** is a holder object that stores a collection of other objects(its elements). They are implemented as class templates, which allows a great flexibility in the types supported as elements. It replicate structures very commonly used in programming : dynamic arrays(vector), queues, stacks, heaps, linked list, trees, maps... We will check three basic containers which are list, tree and table, and for each one we will check two variant Item Based and Model Based.
A **QListWidget** is a convenience class that provides a list view similar to the one supplied by QListView, but with a classic item-based interface for adding and removing items. QListWidget uses an internal model to manage each QListWidgetItem in the list.
The **QTableWidget** class provides an item-based table view with a default model. Table widgets provide standard table display facilities for applications. The items in a QTableWidget are provided by QTableWidgetItem.
The **QTreeWidget class** is a convenience class that provides a standard tree widget with a classic item-based interface similar to that used by the QListView class in Qt 3. This class is based on Qt's Model/View architecture and uses a default model to hold items, each of which is a QTreeWidgetItem.

## ToDoApp
The goal is creating an app to manage tasks. It will have all the features of main application such as menus, actions and toolbar. The application stores an archive of all the pending and finished tasks.
![enter image description here](https://user-images.githubusercontent.com/93097467/150689265-e14b6366-3fa7-4a7c-b0a4-ed24be698618.png)

### Item Based Model

In the Item Based Model, We wrote the code for the graphical and set of actions , now we will write a set of basic functionality. we will start with the connections made for our application:

**First** we add the function for the newTask action,we created a Dialog for the user to add tasks, for that, first we created a Form Class, we use the designer and we obtain the form of AddNew, in addition we added some methods to get the content of our line Edit, checkBox, comboBox and the Date Edit. Here is the Form of Add new task:
![enter image description here](https://user-images.githubusercontent.com/93097467/150689379-727fcb65-16f2-4110-93de-0eb037df9523.png)

**To begin with**, we add those lines to the header file of the dialog class :

   

     class  Dialog1  :  public  QDialog
        
        {
        
          Q_OBJECT
        
        public:
        
          explicit  Dialog1(QWidget  *parent  =  nullptr);
        
          ~Dialog1();
        
          
        
          void  taskDialog();
        
          void  setdate(int  a,  int  m,  int  j);
        
          void  task(QString  t);
        
          void  tag(QString  a);
        
          void  finished(bool  f);
        
          bool  isChecked();
        
          QDate  getDate();
        
          QString  getText();
        
          
        
        private:
        
          Ui::Dialog1  *ui;
        
          
        
        };
    
    
****Then**  in the cpp file we implement our methods:**
   

     Dialog1::Dialog1(QWidget  *parent)  :
    
      QDialog(parent),
    
      ui(new  Ui::Dialog1)
    
    {
    
      ui->setupUi(this);
    
      QDate  date  =QDate::currentDate();
    
      
    
      ui->dateEdit->setMinimumDate(date);
    
      
    
      ui->dateEdit->setDate(date);
    
    }
    
      
    
    Dialog1::~Dialog1()
    
    {
    
      delete  ui;
    
    }
    
    void  Dialog1::setdate(int  a,  int  m,  int  j){
    
      
    
      QDate  d;
    
      
    
      
    
      d.setDate(a,m,j);
    
      
    
      ui->dateEdit->setDate(d);
    
      
    
    }
    
    void  Dialog1::task(QString  t){
    
      ui->lineEdit->setText(t);
    
    }
    
    void  Dialog1::tag(QString  a){
    
      ui->comboBox->setCurrentText(a);
    
    }
    
      
    
    void  Dialog1::finished(bool  f){
    
      ui->checkBox->setChecked(f);
    
      
    
    }
    
    bool  Dialog1::isChecked(){
    
      if(ui->checkBox->isChecked()){
    
      return  true;
    
      }
    
      return  false;
    
    }
    
    QDate  Dialog1::getDate(){
    
      return  ui->dateEdit->date();
    
    }
    
    QString  Dialog1::getText(){
    
      QString  a=  ui->lineEdit->text()+"Due :"  +  ui->dateEdit->text()  +  "Tag :"  +  ui->comboBox->currentText()  +  "";
    
      return  a;
    
    }
**we have declared a private slot called Newtask:**

    private slots:
        void NewTask();
**We created a connexion between the newtask action and the method :**

    connect(newtask,&QAction::triggered,this,&toDoApp::NewTask);
   **Then, we implement the function in the cpp file:**

        void toDoApp::NewTask()
    {
    
        Dialog1 dialog;
        auto replu = dialog.exec();
        if(replu==Dialog1::Accepted){
            QString text = dialog.getText();
            if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
                QIcon TodayIcon(":/Todayicon.png");
    
                ui->pesistent->addItem(new QListWidgetItem(TodayIcon,text));
            }
            else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
                QIcon pendingIcon(":/pending_icon.png");
    
                ui->Pending->addItem(new QListWidgetItem(pendingIcon,text));
            }
            else if(dialog.isChecked()){
                QIcon completedIcon(":/completed_icon.png");
    
                ui->completed->addItem(new QListWidgetItem(completedIcon,text));
            }
        }
    
    }
**We added a closeEvent that save the data in a file.
 First, we declare the slots in the header file:**

     protected:
      void closeEvent(QCloseEvent *e) override;
**Then we implement the function in the cpp file :**


    void toDoApp::closeEvent(QCloseEvent *e){
    
      QFile file("/Users/hp/Desktop/save.txt");
      if (file.open(QIODevice::ReadWrite| QIODevice::Text)){
    
          QTextStream out(&file);
          for (int i=0;i<ui->pesistent->count() ;i++ ) {
              out<< "1"<< ui->pesistent->item(i)->text() << Qt::endl;
          }
          for (int i=0;i<ui->Pending->count() ;i++ ) {
              out<< "2"<< ui->pesistent->item(i)->text() << Qt::endl;
          }
          for (int i=0;i<ui->completed->count() ;i++ ) {
              out<< "3"<< ui->pesistent->item(i)->text() << Qt::endl;
          }
          file.close();
      }
    }
**For open the previous data , we added some line to open our file.txt:**

    QFile file("/Users/hp/Desktop/save.txt");
        if (!file.open(QIODevice::ReadOnly | QIODevice::Text))
            return;
    
        while (!file.atEnd()) {
            QString line = file.readLine();
            if(line.at(0)== "1"){
                ui->pesistent->addItem(line.mid(1,line.size()));
            }else if(line.at(0)== "2"){
                 ui->Pending->addItem(line.mid(1,line.size()));
            }else if(line.at(0)== "3"){
                ui->completed->addItem(line.mid(1,line.size()));
    
            }
        }
  **we added a slots called Pending slot and completed slot First, we declare the slots in the header file :**

      private slots:
        void PendingSlot();
    private slots:
        void CompletedSlot();
    
**Then we implement the function in the cpp file :**

    void toDoApp::PendingSlot(){
        if(ui->Pending->isVisible()){
            ui->Pending->hide();
    }else{
       ui->Pending->show();
        }
    }
    
    void toDoApp::CompletedSlot(){
        if(ui->completed->isVisible()){
            ui->completed->hide();
    }else{
       ui->completed->show();
        }
    }
**We add the connexion of these slots to enable the actions to show or hide our listWidget:**

    connect(pending,&QAction::triggered,this,&toDoApp::PendingSlot);
    connect(completed,&QAction::triggered,this,&toDoApp::CompletedSlot);

 **we added a slots called quit to close the window, and the about and aboutQt slots:**

**First, we declare the slots in the header file:**

    private slots:
        void quit();
        void aboutslot();
        void aboutQtslot();
**Then we implement the functions in the cpp file:**

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
**we added a slot called clear for delete the content of our list views : First, we declare the slots in the header file:**

    private slots:
          void ClearSlot();
**Then we implement the functions in the cpp file:**

    void toDoApp::ClearSlot()
    {
     ui->Pending->clear();
     ui->pesistent->clear();
     ui->completed->clear();
    
     QFile file("/Users/hp/Desktop/save.txt");
     if (file.open(QIODevice::ReadWrite| QIODevice::Text)){
         file.resize(0);
    
     }
    }
**here is the final code of the header file:**

    class toDoApp : public QMainWindow
    {
      Q_OBJECT
    
    public:
      toDoApp(QWidget *parent = nullptr);
     ~toDoApp();
    
      
    private:
      Ui::toDoApp *ui;
    protected:
      void setUpMainWidget();
      void createActions();
      void makeConnexions();
      void createMenus();
      void createToolbars();
      void closeEvent(QCloseEvent *e) override;
      
    private slots:
      void quit();
      void NewTask();
      void aboutslot();
      void aboutQtslot();
      void ClearSlot();
      void Firsttask();
      void secondtask();
      void thirdtask();
    
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
      QStringList Persistent;
      QStringList Pending;
      QStringList Finished;
    
    }
**And here is the implemenation of all functions:**

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
    
        QFile file("/Users/hp/Desktop/save.txt");
        if (!file.open(QIODevice::ReadOnly | QIODevice::Text))
            return;
    
        while (!file.atEnd()) {
            QString line = file.readLine();
            if(line.at(0)== "1"){
                ui->pesistent->addItem(line.mid(1,line.size()));
            }else if(line.at(0)== "2"){
                 ui->Pending->addItem(line.mid(1,line.size()));
            }else if(line.at(0)== "3"){
                ui->completed->addItem(line.mid(1,line.size()));
    
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
    
        ui->Pending->setDragDropMode(QAbstractItemView::DragDrop);
        ui->pesistent->setDragDropMode(QAbstractItemView::DragDrop);
        ui->completed->setDragDropMode(QAbstractItemView::DragDrop);
        ui->Pending->setDefaultDropAction(Qt::MoveAction);
        ui->completed->setDefaultDropAction(Qt::MoveAction);
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
            if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
                QIcon TodayIcon(":/Todayicon.png");
    
                ui->pesistent->addItem(new QListWidgetItem(TodayIcon,text));
            }
            else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
                QIcon pendingIcon(":/pending_icon.png");
    
                ui->Pending->addItem(new QListWidgetItem(pendingIcon,text));
            }
            else if(dialog.isChecked()){
                QIcon completedIcon(":/completed_icon.png");
    
                ui->completed->addItem(new QListWidgetItem(completedIcon,text));
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
        if(ui->completed->isVisible()){
            ui->completed->hide();
    }else{
       ui->completed->show();
        }
    }
    
    void toDoApp::closeEvent(QCloseEvent *e){
    
        QFile file("/Users/hp/Desktop/save.txt");
        if (file.open(QIODevice::ReadWrite| QIODevice::Text)){
    
            QTextStream out(&file);
            for (int i=0;i<ui->pesistent->count() ;i++ ) {
                out<< "1"<< ui->pesistent->item(i)->text() << Qt::endl;
            }
            for (int i=0;i<ui->Pending->count() ;i++ ) {
                out<< "2"<< ui->pesistent->item(i)->text() << Qt::endl;
            }
            for (int i=0;i<ui->completed->count() ;i++ ) {
                out<< "3"<< ui->pesistent->item(i)->text() << Qt::endl;
            }
            file.close();
        }
    }
    
    void toDoApp::ClearSlot()
    {
        ui->Pending->clear();
        ui->pesistent->clear();
        ui->completed->clear();
    
        QFile file("/Users/hp/Desktop/save.txt");
        if (file.open(QIODevice::ReadWrite| QIODevice::Text)){
            file.resize(0);
    
        }
    }
![enter image description here](https://user-images.githubusercontent.com/93097467/150689754-67f68b07-7814-4308-81af-5dd319c5ca84.jpg)
