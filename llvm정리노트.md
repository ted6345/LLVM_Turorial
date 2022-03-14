# LLVM Optimization



## 0. about LLVM

><img src="/home/taehoon/.config/Typora/typora-user-images/image-20220311190724531.png" alt="image-20220311190724531" style="zoom:67%;" />
>
>[컴파일러](https://namu.wiki/w/컴파일러)는 프론트엔드-미들엔드-백엔드의 단계로 구성되어 있다. 보통 이 세 단계는 하나의 프로그램으로 일괄 처리되는데, 이럴 경우 '언어의 종류 x 아키텍처의 종류'만큼 복수의 컴파일러가 필요하게 된다. 이러한 컴파일러 구조는 재사용성을 떨어뜨린다는 문제가 있다. 
>
>프론트엔드가 여러가지 [프로그래밍 언어](https://namu.wiki/w/프로그래밍 언어)들을 중간 표현 코드로 번역하고, LLVM은 그 중간 표현 코드를 각각의 아키텍처에 맞게 최적화하여 실행이 가능한 형태로 바꾸는 방식이다. 처음에는 [GCC](https://namu.wiki/w/GCC)를 프론트엔드로 하고 LLVM은 미들엔드-백엔드로 사용하는 LLVM-GCC 프로젝트(DragonEgg)가 있었으나, LLVM의 자체 프론트엔드인 Clang이 등장한 이후 컴파일의 전 과정을 LLVM 툴체인으로 진행할 수 있게 되었다.
>
>여기서 미들엔드와 백엔드는 LLVM Core라는 [라이브러리](https://namu.wiki/w/라이브러리)가 담당하게 되는데, 각각의 프론트엔드는 사람이 작성한 [소스 코드](https://namu.wiki/w/소스 코드)를 LLVM Core가 인식할 수 있는 중간 코드 LLVM IR(LLVM Intermediate Representation)로 컴파일한다. LLVM IR은 .ll 형식을 가진 LLVM 어셈블리(LLVM Assembly)[[2\]](https://namu.wiki/w/LLVM#fn-2)와 .bc 형식을 가진 LLVM 비트코드(LLVM Bitcode), 그리고 .o 형식을 가진 C++ 목적 코드(C++ Object Code)로 분류된다.
>
>이 중에서 LLVM 어셈블리와 LLVM 비트코드는 둘 다 하드웨어 독립적이고 [어셈블리어](https://namu.wiki/w/어셈블리어)에 가까운 형태를 취하고 있다. 다만 LLVM 어셈블리는 사람이 비교적 읽기 쉬운 원시 코드와 유사한 반면 LLVM 비트코드는 메모리 주소값의 나열로 구성된, 좀 더 최적화가 이루어진 형태의 코드이다. 이러한 LLVM 어셈블리/비트코드는 미들엔드에서 최적화가 한 번 더 진행된 후 백엔드에서 실제 [기계어](https://namu.wiki/w/기계어)로 바뀌어 실행되는데, 그 번역 방식에는 기계어 인터프리트와 실행이 함께 이루어지는 [JIT](https://namu.wiki/w/JIT) 컴파일 방식과 미리 기계어로 다 번역해 놓고 실행하는 AOT 컴파일 방식이 모두 존재한다.
>
>이처럼 각각의 기능들이 분리되면 커다란 장점이 생긴다. 예를 들어 C++ 코드를 이용하여 [x86-64](https://namu.wiki/w/AMD64)와 [ARM](https://namu.wiki/w/ARM(CPU))을 대상으로 바이너리를 만들어야 한다고 가정할 때, 기존의 컴파일러라면 각 아키텍처에 맞는 프론트엔드-미들엔드-백엔드 인프라를 전부 별도로 사용해야 하니 총 2개의 툴체인이 필요하지만, LLVM 툴체인을 사용할 경우 Clang으로 컴파일만 하면 모든 아키텍처에 대한 빌드가 가능해지므로 상당한 편리함을 누릴 수 있다. [윈도우](https://namu.wiki/w/Microsoft Windows) 환경을 정식으로 지원하지 않는 [GCC](https://namu.wiki/w/GCC)나, [유닉스](https://namu.wiki/w/유닉스) 계열 [운영 체제](https://namu.wiki/w/운영 체제)를 정식으로 지원하지 않는 [VC++](https://namu.wiki/w/Visual Studio)와 달리 두 OS를 모두 지원하는 크로스플랫폼 컴파일러라는 것도 장점이다.
>
>LLVM은 컴파일러 개발자에게도 많은 장점이 있는데, 컴파일러를 만드는 쪽에서 프론트엔드부터 백엔드까지 모든 걸 구현해야 했던 예전과는 달리, 프론트엔드 및 LLVM과의 연결부만 구현하면 LLVM이 코드 최적화와 기계어 생성을 담당해 주기 때문이다. 덕분에 현재 다양한 언어에 대응하는 여러 LLVM 프론트엔드 컴파일러들이 만들어지고 있다.







## Step1

- clang, llvm-as, llvm-dis, llc 등의 툴을 사용하여 소스 코드 (*.c)를 IR 코드 (*.bc, *.ll)로, 그리고 다시 바이너리로 컴파일하는 방법을 익힙니다

​		c -> bc , ll

- LVM의 API를 사용하여 이렇게 컴파일 된 IR 코드를 컴파일러 프로그램에서 (수정할 수 있도록) 로드 합니다.



###  1.1 clang

---

LLVM IR 기반 컴파일러로, C, C++등 다양한 프론트 엔드들을 지원합니다.

 –I, -l, -L, -W, -c, -g 등 GCC에서도 통용되는 일반적인 컴파일 옵션을 대부분 지원.



-S 옵션 : cc1으로 전처리된 파일을 어셈블리 파일로 컴파일까지만 수행하고 멈춘다. (\*.s)

-c 옵션 : as에 의한 어셈블까지만 수행하고 링크는 수행하지 않는다.

---

• Clang을 사용하여 HelloWorld.c 파일을 컴파일하고 실행합니다.
$ clang <source_file> [-o output_path]

```
clang HelloWorld.c -o HelloWorld
./HelloWorld
```



---

• Clang을 사용하여 소스코드를 LLVM IR 형태로 컴파일 할 수 있습니다.

$ clang –emit-llvm -S <source_file> [-o output_path] # readable (*.ll)
$ clang –emit-llvm -c <source_file> [-o output_path]

```
clang -emit-llvm -S HelloWorld.c -o HelloWorld.ll
clang -emit-llvm -c HelloWorld.c -o HelloWorld.bc
```



###  1.2 llvm-as , llvm-dis

---

IR 레벨의 Assembler와 Disassembler로 이를 사용하여 human-readable assembly 포맷과 bitcode 포맷의 IR 코드를 각각 다른 문법으로 바꿀 수 있습니다. 단, 파일에 (IR 문법 등의 이유로) 오류가 있다면 이
작업을 수행 할 수 없습니다.

$ llvm-as <IR file> [-o=output path]  # readable (*.ll) à bitcode (*.bc)
$ llvm-dis <IR file> [-o=output path] # bitcode(*.bc) à readable (*.ll)



```
llvm-as Helloworld.ll -o=HelloWorld.2.bc
llvm-dis Helloworld.bc -o =HelloWorld.2.ll
```





###  1.3 llc

---

LLVM backend compile로 bitcode 포맷의 LLVM IR을 머신 어셈블리로 변환합니다.

$ llc <IR file> [-o output_path]

```
llc HelloWorld.ll -o HelloWorld.s
```

clang을 사용하면 IR에서 실행가능한 바이너리로 컴파일 할 수 있습니다.
(llc로 IR코드를 어셈블리로 컴파일하고 어셈블리를 돌린 후 링킹해서 실행가능한 바이너리를 얻을 수도 있습니다.)

```
clang HelloWorld.ll -o Helloworld
```



### 1.4 llvm-config

---

• LLVM 에서 제공하는 라이브러리 및 인터페이스들을 통해 LLVM IR을 명령어 수준에서 처리 할 수 있습니다.
• 이러한 LLVM IR을 처리하는 프로그램에서 LLVM 라이브러리들을 사용하기 위해, LLVM은 통합된 정보를 제공하기 위
한 바이너리인 llvm-config를 제공합니다. llvm-config에서는 다양한 경로, 특정 기능을 사용하기 위해 필요한 종속 라
이브러리 이름 등을 제공합니다.

$ clang++ <file> $(llvm-config --cxxflags --ldflags --system-libs --libs [name list])



• clang을 사용하여 IR을 읽어서 모듈이름을 출력하는 ReadIR.cpp를 컴파일합니다.

```
clang++ ReadIR.cpp -o ReadIR $(llvm-config --cxxflags --ldflags --system-libs --libs mcjit irreader)
```

• ReadIR을 실행하여 HelloWorld.ll과 HelloWorld.bc을 컴파일러 프로그램(ReadIR) 내에서 로드하고 모듈 이름을 출력
합니다

```
./ReadIR HelloWorld.ll
./ReadIR HelloWorld.bc
```

• ReadIR.cpp 내의 llvm::Module 클래스는 컴파일 및 최적화를 수행하기 위한 단위를 나타내는 클래스로 보통 하나의
파일을 의미합니다. llvm::LLVMContext 클래스는 전체 컴파일과정에서 일관된 자료형 및 전역 변수 등을 포함하는
컨택스트를 나타냅니다.





##  Step 2 - IR 내의 명령어들을 C++ iterator를 사용하여 순회 및 출력

>• LLVM의 기본적인 **Module 구조를 이해하고, IR 내의 명령어들을 C++ iterator를 사용하여 순회 및 출력**합니다.
>
>• LLVM IR에서는 **llvm::Module** -> **llvm::Function** -> l**lvm::BasicBlock** -> **llvm::Instruction**의 계층 구조로 타겟 프로그램을 관리합니다.
>
>•LLVM의 모델에 따르면 프로그램은 다음과 같은 구조로 되어있습니다.
>Module은 여러 개의 Function로 구성되어 있고,
>Function은 다시 여러 개의 Basic Block 으로 구성되어 있으며,
>Basic Block은 Instruction로 구성되어 있다.
>– Module = 모듈, 일반적으로 하나의 소스 파일
>– Function = 함수
>– Basic Block = Straight Forward Code Section, Branch나 Return같은 제어 명령어로 끝남
>– Instruction = 명령어
>
>• 전역 변수를 의미하는 llvm::GlobalVariable 등은 llvm::Module 내에서 따로 관리 됩니다.
>• 각 클래스들의 Iterator또는 다른 메소드들을 사용하여 위 계층구조를 접근 할 수 있습니다.
>• Step 2에서는 Step 1에서 읽은 IR 코드의 함수와 명령어들을 출력합니다. 출력을 하기 위해서 llvm::raw_os_ostream 인스턴스를 사용합니다.
>
>![image-20220312114253701](/home/taehoon/.config/Typora/typora-user-images/image-20220312114253701.png)

![image-20220314105259876](/home/taehoon/.config/Typora/typora-user-images/image-20220314105259876.png)

### 2.2 printinst.cpp

```c++
void ParseIRSource(void);
void TraverseModule(void);

int main(int argc , char** argv)
{
        if(argc < 2)
        {
                std::cout << "Usage: ./PrintInst <ir_file_path>" << std::endl;
                return -1;
        }
        input_path = std::string(argv[1]);

        // Read & Parse IR Source
        ParseIRSource();
        // Traverse TheModule
        TraverseModule();

        return 0;
}
// Read & Parse IR Sources
//  Human-readable assembly(*.ll) or Bitcode(*.bc) format is required
void ParseIRSource(void)
{
        llvm::SMDiagnostic err;

        // Context
        TheContext = new llvm::LLVMContext();
        if( ! TheContext )
        {
                std::cerr << "Failed to allocated llvm::LLVMContext" << std::endl;
                exit( -1 );
        }

        // Module from IR Source
        TheModule = llvm::parseIRFile(input_path, err, *TheContext);
        if( ! TheModule )
        {
                std::cerr << "Failed to parse IR File : " << input_path << std::endl;
                exit( -1 );
        }
}

// Traverse Instructions in TheModule
void TraverseModule(void)
{
        llvm::raw_os_ostream raw_cout( std::cout );

        // Module::iterator --> Function
        for( llvm::Module::iterator ModIter = TheModule->begin(); ModIter != TheModule->end(); ++ModIter )
        {
                llvm::Function* Func = llvm::cast<llvm::Function>(ModIter);

                // Print Function Name
                raw_cout << Func->getName() << '\n';

                // Function::iterator --> BasicBlock
                for( llvm::Function::iterator FuncIter = Func->begin(); FuncIter != Func->end(); ++Fu
ncIter )
                {
                        llvm::BasicBlock* BB = llvm::cast<llvm::BasicBlock>(FuncIter);

                        // BasicBlock::iterator --> Instruction
                        for( llvm::BasicBlock::iterator BBIter = BB->begin(); BBIter != BB->end(); ++
BBIter )
                        {
                                llvm::Instruction* Inst = llvm::cast<llvm::Instruction>(BBIter);

                                // Print Instruction
                                raw_cout << '\t';
                                Inst->print(raw_cout);
                                raw_cout << '\n';
                        }
                }
        }
}


```









## Step3 -프로그램 내 특정 명령어의 수를 세어 출력하는 간단한 정적 프로파일러

>• Step3에서는 프로그램 내 **특정 명령어의 수를 세어 출력하는 간단한 정적 프로파일러**를 만듭니다.
>
>• LLVM 6.0.1 기준으로, LLVM IR에는 총 64개의 명령어 Opcode가 있습니다.
>LLVM은 llvm::CallInst로 표현되는 Intrinsic이나 Vector 명령어 등 보다 더 많은 명령어들을 표현하고 있습니다.
>
>• llvm/include/llvm/IR/Instruction.def
>자주 사용되는 몇가지 명령어들은 다음과 같습니다
>– BinaryOperator: add, sub, mul 과 같은 산술 연산이나, and, or과 같은 비교 연산 등 2개의 operand들을 연산하는 명령어들
>– ReturnInst, BranchInst 등: 제어 흐름관련 명령어들
>– CallInst: 함수 호출 명령어
>– CastInst: 타입 변환 명령어
>– AllocaInst: 스택(정적)에 메모리 할당하는 명령어
>– LoadInst, StoreInst: 메모리에 있는 데이터들을 접근하는 명령어
>– GetElementPtrInst: 배열 접근 등에서 메모리 접근하는 명령어



#### 3.1 isBinaryOp()

>• LLVM 6 기준 Add 명령어는 llvm::Instruction 클래스의 자식 클래스인 llvm::BinaryOperator 자료형으로 포함됩니다.
>이전 슬라이드처럼 llvm::Instruction의 멤버 함수 isBinaryOp()를 사용하여 이를 확인 할 수 있습니다.
>• 하지만 Add 명령어는 BinaryOpeartor의 Opcode로만 나타내어지기 때문에 (Add만을 위한 특별한 클래스가 없고, Sub, Mul등과 같이 BinaryOperator로 표현됨) 이 방법으로 해당 명령어가 Add 명령어인지는 확신 할 수 없습니다
>
>![image-20220314111154342](/home/taehoon/.config/Typora/typora-user-images/image-20220314111154342.png)

#### 3.2 

>- llvm::Instruction의 getOpcode() 메소드를 사용하여 Opcode를 직접 얻어서 현재 명령어가 어떤 명령어인지 확인 할수 있습니다.
>
>![image-20220314111253806](/home/taehoon/.config/Typora/typora-user-images/image-20220314111253806.png)

![image-20220314111942876](/home/taehoon/.config/Typora/typora-user-images/image-20220314111942876.png)



#### 3.3 -O3 옵션

>• -O3 옵션을 적용하면, Loop Unrolling과 같은 최적화가 적용되므로, 적용 전 후의 ADD 명령어 수가 다릅니다.
>• 다른 종류의 명령어들 또한 같은 방법으로 카운팅 할 수 있습니다.

![image-20220314112131667](/home/taehoon/.config/Typora/typora-user-images/image-20220314112131667.png)



#### 3.4 Exercise

– Exercise 1: add, sub, mul, div의 개수를 모두 카운팅 합니다.

```
if( Inst->getOpcode() == llvm::Instruction::Add ) { total_add_inst++; }
if( Inst->getOpcode() == llvm::Instruction::Sub ) { total_sub_inst++; }
if( Inst->getOpcode() == llvm::Instruction::Mul ) { total_mul_inst++; }
if( Inst->getOpcode() == llvm::Instruction::SDiv ) { total_div_inst++; }
```

![image-20220314125011595](/home/taehoon/.config/Typora/typora-user-images/image-20220314125011595.png)

– Exercise 2: 어떤 함수가 몇 번씩 호출되었는지 정적으로 카운팅 합니다.
(Hint: llvm::CallInst 에서, getCalledFunction()을 통해 호출되는 함수를 접근 할 수 있습니다.)

```
map<StringRef,int> m;
	
	...
	StringRef name = Func->getName();
		if(m.find(name) == m.end()){
			m.insert({name,1});
		}
		else{
			int tmp = m.find(name)->second;
			m.find(name)->second = tmp+1;
		}
	...

map<StringRef,int>::iterator itr;
	for(itr = m.begin(); itr!=m.end(); ++itr){
	
		cout<< "Function Name: " << (itr->first).data() <<" , Count :"<<itr->second<<endl;
	}

```

![image-20220314143900403](/home/taehoon/.config/Typora/typora-user-images/image-20220314143900403.png)





## Step4. IR 레벨에서 프로그램 내의 명령어를 삭제하거나 새로 생성하여 삽입

>• Step 4에서는 IR 레벨에서 프로그램 내의 명령어를 삭제하거나 새로 생성하여 삽입합니다.
>• Step 3을 바탕으로 전체 명령어 중 특정 종류의 명령어에 접근 할 수 있습니다. 이를 바탕으로 Step 4에서는 
>
>(1) 새로운 명령어를 생성하고 (2) 기존 명령어를 삭제하고 (3) 명령어간 종속성에 문제가 없도록 새로운 명령어로 기존 명령어를 대체하는 과정을 수행합니다.



#### 4.1 새로운 명령어 생성

• LLVM에서는 명령어를 생성하기 위해 llvm::IRBuilder 클래스를 제공하고 있습니다. 이 IRBuilder 인스턴스를 사용할 수
있고, 각 명령어 클래스들에서 제공하는 static 메소드 Create를 사용하여서 명령어를 생성 할 수도 있습니다.
• 일반적으로 LLVM은 아래와 같이 수행할 명령어의 Operand들을 Create 메소드의 인자로 요구합니다. Operand에는
각 명령어 별로 요구하는 것에 따라 Value, Type 등이 될 수 있습니다.

![image-20220314150507101](/home/taehoon/.config/Typora/typora-user-images/image-20220314150507101.png)

•또한 결과 값을 가지는 명령어 (binary operator, non-void return function call 등) 또한 Instruction 객체(인스턴스)를 다른 Isntruction의 Operand로 사용함으로써 그 결과 값을 다른 명령어의 Operand로 사용할 수 있습니다.
(llvm::Instruction 클래스는 llvm::Value 클래스의 자식 클래스입니다.)



•InsertInst는 타겟 어플리케이션 내 ADD 명령어가 있다면 바로 그 앞에 1 + 1연산을 추가합니다.
(즉 ADD 1, 1의 명령어를 삽입합니다)

![image-20220314150825507](/home/taehoon/.config/Typora/typora-user-images/image-20220314150825507.png)



• 타겟 어플리케이션과 작성한 프로그램을 각각 컴파일하고, 수행 후 원본과 처리 후 파일을 비교합니다.

![image-20220314152138579](/home/taehoon/.config/Typora/typora-user-images/image-20220314152138579.png)



![image-20220314152108650](/home/taehoon/.config/Typora/typora-user-images/image-20220314152108650.png)

• IR 레벨에서 명령어를 추가한 타겟 어플리케이션을 바이너리로 컴파일하고 실행합니다.
(IR에 오류가 있다면 바이너리로 컴파일 되지 않습니다)

![image-20220314152210204](/home/taehoon/.config/Typora/typora-user-images/image-20220314152210204.png)





#### 4.2 특정명령어 삭제

• 특정 명령어를 삭제하기 위해서는, 당연히 해당 명령어의 결과 값을 사용하는 명령어들의 종속 관계를 해결해 주어야
합니다. 

예를 들어 A = 2 + 3를 계산하는 연산에서  i) 명령어가 2 + 3을 연산하는 명령어 1번과 명령어  ii) 1번의 결과 값을
A에 저장(Store) 하는 명령어 2가가 있다고 가정할 때, 명령어 1의 결과 값이 명령어 2에서 사용되므로 명령어 1을 지
우면 컴파일을 제대로 수행할 수 없습니다.

• 따라서 이 Step4에서는 타겟 어플리케이션 내에 있는 ADD 명령어들을 SUB 명령어로 바꾸는 과정을 수행하는데, 이
를 위해 (1) ADD 명령어와 같은 피연산자들로 SUB 명령어를 생성하고 (2) ADD 명령어의 결과 값이 사용되는 곳(USE)
들을 SUB 명령어의 결과를 사용하도록 변경하고, (3) 최후에 ADD 명령어를 삭제하는 과정을 거칩니다.

•IR 레벨의 소스에서 생기는 문제는 주로 Human-readable IR로 현재의 Module을 출력한 이후, llvm-as를 통해
bitcode로 포맷을 변환하면 확인 할 수 있습니다





#### 4.2.1 ADD 명령어 이전에 SUB 명령어가 수행

![image-20220314152817226](/home/taehoon/.config/Typora/typora-user-images/image-20220314152817226.png)



#### 4.2.2 Sub 명령어가 Add 명령어가 사용되는 곳에 대신하여 사용

![image-20220314152905959](/home/taehoon/.config/Typora/typora-user-images/image-20220314152905959.png)

#### 4.2.3 ADD 명령어를 삭제합니다

> 단, 현재 BBIter가 삭제할 명령어를 가리키고 있기 때문에, loop 내에서 삭제하면 오류
> 가 발생합니다. loop 밖에서 삭제해야 합니다.

![image-20220314152948153](/home/taehoon/.config/Typora/typora-user-images/image-20220314152948153.png)

#### 4.2.4 컴파일

• 타겟 어플리케이션과 작성한 프로그램을 각각 컴파일하고, 수행 후 원본과 처리 후 파일을 비교합니다.

![image-20220314153332510](/home/taehoon/.config/Typora/typora-user-images/image-20220314153332510.png)

![image-20220314153409265](/home/taehoon/.config/Typora/typora-user-images/image-20220314153409265.png)

• IR 레벨에서 명령어를 추가한 타겟 어플리케이션을 바이너리로 컴파일하고, 실행합니다.

![image-20220314153517297](/home/taehoon/.config/Typora/typora-user-images/image-20220314153517297.png)





#### 4.3 Exercise

– Exercise 1: 앞서의 Step 4에서 (2)의 코드를 삭제한 이후 ((3)은 수행), 어떤 문제가 발생하는지 확인합니다.

![image-20220314153812045](/home/taehoon/.config/Typora/typora-user-images/image-20220314153812045.png)



– Exercise 2: A + B + C의 연산 패턴을 탐색하고, 이를 A * B - C로 변경합니다.
(Hint: 연산 순서상, A + B + C는 (A + B) + C와 같은데, 이것은 A + B 명령어의 결과 값이 ADD 명령어의
Operand로 들어가는 것과 같습니다.)

![image-20220314154852405](/home/taehoon/.config/Typora/typora-user-images/image-20220314154852405.png)

![image-20220314154817838](/home/taehoon/.config/Typora/typora-user-images/image-20220314154817838.png)

![image-20220314154616507](/home/taehoon/.config/Typora/typora-user-images/image-20220314154616507.png)

![image-20220314154830909](/home/taehoon/.config/Typora/typora-user-images/image-20220314154830909.png)





## Step 5 . IR 레벨에서 명령어의 위치를 변경하고, 메모리 명령어의 종속성 관계

>• Step 5에서는 IR 레벨에서 명령어의 위치를 변경하고, 메모리 명령어의 종속성 관계에 대해서 파악합니다.
>
>• 일반적인 산술 연산 명령어와 다르게, 메모리 명령어 (load/store)의 종속 관계는 컴파일러 입장에서 대부분 알기 어렵습니다. 
>
>예를 들어 주소 X에서 값을 읽은 명령어와(load) 주소 Y에 값을 저장하는 명령어(store)가 있다고 할 때, 주소
>X와 주소 Y가 같거나, 다르다는 보장이 없는 이상 두 명령어간 순서는 두 명령어간 def-use chain에서 종속 관계가 없
>다고 하더라도 쉽게 바뀔 수 없습니다.
>
>•Step 5에서는 Store 명령어를 Store 명령어가 있는 BasicBlock 가장 마지막 부분으로 위치를 옮기는 작업을 수행합니다.
>
>•앞서 설명한 것의 이유로, Store 명령어의 위치 이동으로 이루어지는 종속성 문제는 컴파일러 레벨에서 특별한 오류
>를 발생시키지 않습니다.
>
>
