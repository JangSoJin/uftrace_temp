* uftrace - Function graph tracer for userspace
* uftrace.c 의 main 부분을 간략하게 분석

편의상 아래의 구조체에서 mode, color, options 만 설명하도록 한다.

~~~
struct opts opts = {
		.mode		= UFTRACE_MODE_INVALID,
		.dirname	= UFTRACE_DIR_NAME,
		.libcall	= true,
		.bufsize	= SHMEM_BUFFER_SIZE,
		.depth		= OPT_DEPTH_DEFAULT,
		.max_stack	= OPT_RSTACK_DEFAULT,
		.port		= UFTRACE_RECV_PORT,
		.use_pager	= true,
		.color		= COLOR_AUTO,  /* default to 'auto' (turn on if terminal) */
		.column_offset	= 8,
		.comment	= true,
		.kernel_skip_out= true,
		.fields         = NULL,
		.sort_column	= 2,
		.event_skip_out = true,
	};
struct argp argp = {
		.options = uftrace_options,
		.parser = parse_option,
		.args_doc = "[record|replay|live|report|info|dump|recv|graph|script] [<program>]",
		.doc = "uftrace -- function (graph) tracer for userspace",
};
~~~

우선, 전체적인 사용법은 argp 구조체에서 args_doc 을 통해 파악할 수 있듯,

**`uftrace [record|replay|live|report|info|dump|recv|graph|script] [options] COMMAND [command-options]`** 를 통해 사용이 가능하다.

**color**는 출력 (replay) 시에 보여지기 위한 enable, disable을 지정할 수 있는 옵션
 사용은 아래와 같다
 
  `uftrace replay --color=yes`

**mode**는 SUB-COMMANDS (record, replay, live, report, info, dump, recv, graph, script) 중에

~~~
switch (opts.mode) {
	case UFTRACE_MODE_RECORD:
		ret = command_record(argc, argv, &opts);
		break;
	case UFTRACE_MODE_REPLAY:
		ret = command_replay(argc, argv, &opts);
		break;
	case UFTRACE_MODE_LIVE:
		ret = command_live(argc, argv, &opts);
		break;
	case UFTRACE_MODE_REPORT:
		ret = command_report(argc, argv, &opts);
		break;
	case UFTRACE_MODE_INFO:
		ret = command_info(argc, argv, &opts);
		break;
	case UFTRACE_MODE_RECV:
		ret = command_recv(argc, argv, &opts);
		break;
	case UFTRACE_MODE_DUMP:
		ret = command_dump(argc, argv, &opts);
		break;
	case UFTRACE_MODE_GRAPH:
		ret = command_graph(argc, argv, &opts);
		break;
	case UFTRACE_MODE_SCRIPT:
		ret = command_script(argc, argv, &opts);
		break;
	case UFTRACE_MODE_INVALID:
		ret = 1;
		break;
	}
~~~

위와 같은 switch 구문에서 처리한다. 예를 들어 opts.mode가 UFTRACE_MODE_REPLAY 인 경우 command_replay 함수를 실행시키고 
command_replay 는 cmd-replay.c 에 선언되어있다.

* 우선 예제 c 파일 hello.c 생성 후, `gcc -pg hello hello.c` 로 컴파일

~~~
#include <stdio.h>

int bar(int b){
	printf(“bar %d\n”,b);
	return b;
}

int foo(int a){
	printf(“foo %d\n”,a);
	return bar(a+1);
}

void main(int arg, char *argv[]){
	printf(“hello %d\n”,foo(arg));
}
~~~

간략하게 **SUB-COMMANDS**를 설명하자면,

~~~
* record : data file 혹은 directory 내에 trace data를 저장한다.

* replay : record 된 후의 trace data를 time duration과 함께 출력한다.

* live : live tracing 한다. 

* report : record 된 data를 Total time, Self time, Call된 횟수, Function 과 같은 통계적, 요약적 정보를 통해 나타낸다.

* info : system 정보와 process 정보로 나누어 출력하여준다. ( system의 경우 cpu 정보나 프로그램 버전, process의 경우 cpu time, context switch와 같은 정보 출력)

* dump : raw tracing data 출력

* recv : network로부터 data를 받아서 저장

* graph : function call graph 를 출력

* script : run a script
~~~

SUB-COMMANDS 뒤에 다음과 같은 **OPTIONS**도 줄 수 있다.

~~~
* -?, --help : 도움말 출력

* --usage : 사용법 출력

* -V, --version : program version 출력

* -v, --verbose : verbose messages 형태 출력

* --debug : debug message 출력 (debug-domain=DOMAIN 과 같이 도메인(uftrace,symbol,demangle,filter,fstack, session, kernel, mcount, dynamic, event)을 설정하여서도 사용 가능)

* --no-pager : pager 사용 안함

* --color=yes/no/auto 중 하나 : color enable, disable 설정
~~~
