# 组件协作

## 模板方法 

设计模式的要点是“寻找变化点，然后在变化点处应用设计模式，从而来更好地应对需求的变化”



**重构关键技法**

静态——> 动态

早绑定 ——> 晚绑定

继承 ——>组合

编译时依赖 ——> 运行时依赖

紧耦合 ——> 松耦合



框架(liberty)与应用程序（application）的划分，“组件协作”模式通过晚期绑定，来实现框架与应用程序之间的松耦合

对于某一项任务，它常常有**稳定**的整体操作结构，但各个子步骤却有很多**改变**的需求



结构化设计

![image-20200809105807188](https://raw.githubusercontent.com/raylee-lilei/PicGoImage/master/imgimage-20200809105807188.png)

面向对象设计

![image-20200809105915443](https://raw.githubusercontent.com/raylee-lilei/PicGoImage/master/imgimage-20200809105915443.png)

```C++
//程序库开发人员
class Library{

public:
	void Step1(){
		//...
	}

    void Step3(){
    	//...
    }
    
    void Step5(){
    	//...
    }

};

//应用程序开发人员
class Application{
public:
	bool Step2(){
		//...
    }

    void Step4(){
		//...
    }
};

int main()
{
	Library lib();
	Application app();

	lib.Step1();

	if (app.Step2()){
		lib.Step3();
	}

	for (int i = 0; i < 4; i++){
		app.Step4();
	}

	lib.Step5();

}

引用程序开发人员完成主流程，实现早绑定

```



```C++
//程序库开发人员
class Library{
public:
	//稳定 template method
    void Run(){
        
        Step1();

        if (Step2()) { //支持变化 ==> 虚函数的多态调用
            Step3(); 
        }

        for (int i = 0; i < 4; i++){
            Step4(); //支持变化 ==> 虚函数的多态调用
        }

        Step5();

    }
	virtual ~Library(){ }

protected:
	
	void Step1() { //稳定
        //.....
    }
	void Step3() {//稳定
        //.....
    }
	void Step5() { //稳定
		//.....
	}

	virtual bool Step2() = 0;//变化
    virtual void Step4() =0; //变化
};


//应用程序开发人员
class Application : public Library {
protected:
	virtual bool Step2(){
		//... 子类重写实现
    }

    virtual void Step4() {
    	//... 子类重写实现
    }

};

int main()
	{
	    Library* pLib=new Application();
	    lib->Run();

		delete pLib;
	}

}

库函数开发人员写好主流程框架（run），并且在protected 里将变化的函数定义成虚函数，让子类去重写变化的函数。应用程序开发人员完成对变化函数的重写。晚绑定
```

模板方法用最简洁的机制（虚函数的多态性）为很多应用程序框架提供了灵活的扩展点，对于lib 开发人员而言，不要调用我，让我来调用你。被Template Method调用的虚方法可以具有实现，也可以没有任何实现（抽象方法、纯虚方法），但一般推荐将它们设置为protected方法。



## 策略模式

**使用扩展的方式来应对变化**

某些对象使用的算法可能多种多样，经常改变，如果将这些算法都编码到对象中，将会使对象变得异常复杂

比如每个国家都有计算税收的 不同算法。

```C++
enum TaxBase {
	CN_Tax,
	US_Tax,
	DE_Tax,
	FR_Tax       //更改    当我增加一个法国的税收，需要来更改，而不是扩展
};

class SalesOrder{
    TaxBase tax;
public:
    double CalculateTax(){
        //...
        

        if (tax == CN_Tax){
            //CN***********
        }
        else if (tax == US_Tax){
            //US***********
        }
        else if (tax == DE_Tax){
            //DE***********
        }
    	else if (tax == FR_Tax){  //更改  当我增加一个法国的税收，需要来更改，而不是扩展
    		//...
    	}
    
        //....
     }

};
```

策略模式

```C++
class TaxStrategy{
public:
    virtual double Calculate(const Context& context)=0;
    virtual ~TaxStrategy(){}
};


class CNTax : public TaxStrategy{
public:
    virtual double Calculate(const Context& context){
        //***********
    }
};

class USTax : public TaxStrategy{
public:
    virtual double Calculate(const Context& context){
        //***********
    }
};

class DETax : public TaxStrategy{
public:
    virtual double Calculate(const Context& context){
        //***********
    }
};



//扩展
//*******************************************************************
class FRTax : public TaxStrategy{
public:
	virtual double Calculate(const Context& context){
		//.........
	}
};
//********************************************************************


class SalesOrder{
private:
    TaxStrategy* strategy;

public:
    SalesOrder(StrategyFactory* strategyFactory){
        this->strategy = strategyFactory->NewStrategy();
    }
    ~SalesOrder(){
        delete this->strategy;
    }

    public double CalculateTax(){
        //...
        Context context();
        
        double val = 
            strategy->Calculate(context); //多态调用  运行时调用不同的算法
        //...
    }

};
```



在运行时方便地根据需要在各个算法之间进行切换

Strategy模式提供了用条件判断语句以外的另一种选择，消除条件判断语句，就是在解耦合。含有许多条件判断语句的代码通常都需要Strategy模式

有时候支持不使用的算法也是一个性能负担（如果代码装载到中国，使用if else的使得被转载在内存的其他else if 永远不会执行，浪费内存，不能节省开销）



## 观察者模式