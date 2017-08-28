---
layout: post
title: 一些设计模式总结
category: 技术
tags: designPattern
keywords: 
description: 
---


## 工厂模式
### 普通工厂模式
由一个工厂类根据传入的参数，动态决定应该创建哪一个产品类。

例：一个打印工厂可以实现条码打印和二维码打印

``` java

/**
 * 打印接口
 * @author yyf
 */
public interface Print {
	public void print();
}

/**
 * 条码打印机
 * @author yyf
 */
public class BarcodePrint implements Print{
	@Override
	public void print() {
		System.out.println("BARCODE PRINT!");
	}
}


/**
 * 二维码打印机
 * @author yyf
 */
public class QrcodePrint implements Print {
	@Override
	public void print() {
		System.out.println("QRCODE PRINT!");
	}
}

/**
 * 打印工厂
 * @author yyf
 */
public class PrintFactory {
//	public Print print(String type) {
//		return type.equals("BARCODE")? new BarcodePrint(): new QrcodePrint(); 
//	}
	public Print BarPrint(){
		return new BarcodePrint();
	}
	public Print QrPrint(){
		return new QrcodePrint();
	}
}

/**
 * 测试类
 * @author yyf
 *
 */
public class Test {
	public static void main(String[] args) {
		PrintFactory p = new PrintFactory();
//		p.print("QRCODE").print();
		p.QrPrint();
	}
}

``` 
 
### 抽象工厂模式
由一个工厂类根据传入的参数，动态决定应该创建哪一个产品类。
由一个工厂=类创建多个产品族，在产品族中决定创建某一个产品类

例：一个打印工厂可以实现东芝打印机接口和斑马打印机接口，东芝打印机可以打印条形码和二维码，斑马打印机同理，在这里打印机属性是族（产品族），打印机打印功能是属性类（产品等级）。

``` java

/**
 * 打印接口
 * @author yyf
 */
public interface Print {
	public Qrcode printQrcode();
	public Barcode printBarcode();
}

/**
 * 条码打印机接口
 * @author yyf
 */
public interface  Barcode {

}

/**
 * 二维码打印机接口 
 * @author yyf
 */
public interface  Qrcode  {
}

/**
 * 东芝打印接口
 * @author yyf
 *
 */
public class ToshibaPrint implements Print{

	@Override
	public Barcode printBarcode() {
		return new ToshibaBarcode();
	}

	@Override
	public Qrcode printQrcode() {
		return new ToshibaQrcode();
	}
}

/**
 * 东芝条码打印类
 * @author yyf
 *
 */
public class ToshibaBarcode implements Barcode{
    public ToshibaBarcode() {
        System.out.println("Barcode工厂创建ToshibaBarcode");
    }
}

/**
 * 东芝二维码打印类
 * @author yyf
 *
 */
public class ToshibaQrcode implements Qrcode {
	public ToshibaQrcode() {
		System.out.println("Qrcode工厂创建ToshibaBarcode");
	}
}

/**
 * 斑马打印接口
 * @author yyf
 *
 */
public class ZebarPrint implements Print {

	@Override
	public Barcode printBarcode() {

		return new ZebarBarcode();
	}

	@Override
	public Qrcode printQrcode() {

		return new ZebarQrcode();
	}
}

/**
 * 斑马条码打印类
 * @author yyf
 *
 */
public class ZebarBarcode implements Barcode{
    public ZebarBarcode() {
        System.out.println("Barcode工厂创建ZebarBarcode");
    }
}

/**
 * 斑马二维码打印类
 * @author yyf
 *
 */
public class ZebarQrcode implements Qrcode {
	public ZebarQrcode() {
		System.out.println("Qrcode工厂创建ZebarQrcode");
	}
}

/**
 * 测试类
 * @author yyf
 * 
 */
public class Test {
	public static void main(String[] args) {
		Print toshi = new ToshibaPrint();
		Print zebar = new ZebarPrint();
		toshi.printBarcode();
		toshi.printQrcode();
		zebar.printBarcode();
		zebar.printQrcode();
	}
}

输出：Barcode工厂创建ToshibaBarcode
	 Qrcode工厂创建ToshibaBarcode
	 Barcode工厂创建ZebarBarcode
	 Qrcode工厂创建ZebarQrcode

``` 

##策略模式
策略模式属于对象的行为模式。针对一组算法，将每一个算法封装到具有共同接口的独立的类中，从而使得它们可以相互替换。
这个模式涉及到三个角色：
环境(Context)角色：持有一个Strategy的引用。
抽象策略(Strategy)角色：这是一个抽象角色，通常由一个接口或抽象类实现。此角色给出所有的具体策略类所需的接口。
具体策略(ConcreteStrategy)角色：包装了相关的算法或行为。

例：会员制度，高级会员打77折，中级会员88折，普通会员99折。

``` java

/**
 * 抽象策略(Strategy)角色
 * @author yyf
 * 
 */
public interface MemberStrategy {
	public double getPrice(double price);
}

/**
 * 初级会员策略 99折
 * @author yyf
 */
public class PrimaryMemberStrategy  implements MemberStrategy{
	@Override
	public double getPrice(double price) {
		return 0.99 * price;
	}
}

/**
 * 中级会员策略 88折
 * @author yyf
 */
public class IntermediateMemberStrategy implements MemberStrategy {
	@Override
	public double getPrice(double price) {
		return 0.88 * price;
	}
}

/**
 * 高级会员策略 77折
 * @author yyf
 */
public class AdvancedMemberStrategy implements MemberStrategy{
	@Override
	public double getPrice(double price) {
		return 0.77 * price;
	}
}

/**
 * 环境(Context)角色
 * @author yyf
 */
public class MemberContext {
	private MemberStrategy strategy;
	public MemberContext(MemberStrategy strategy) {
		this.strategy = strategy;
	}
	public double getPrice(double price) {
		return this.strategy.getPrice(price);
	}
}

/**
 * 测试类
 * @author yyf
 */
public class Test {
	public static void main(String[] args) {
		double price = 100.0; //原价100
		System.out.println("初级会员折后价格：" + new MemberContext(new PrimaryMemberStrategy()).getPrice(price));
		System.out.println("中级会员折后价格：" + new MemberContext(new IntermediateMemberStrategy()).getPrice(price));
		System.out.println("高级会员折后价格：" + new MemberContext(new AdvancedMemberStrategy()).getPrice(price));
	}
}

输出：初级会员折后价格：99.0
	 中级会员折后价格：88.0
	 高级会员折后价格：77.0

``` 







	





