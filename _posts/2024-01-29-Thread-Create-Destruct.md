---
title: "스레드의 생성과 종료(_beginthreadex, _endthreadex 소스코드 분석)"
date: "2024-01-29 22:07:00 +0900"
last_modified_at: "2024-01-29 22:07:00 +0900"
categories:
    - 멀티스레드

tags:
    - [멀티스레드, C++]

toc: true
excerpt: "stl에서 제공하는 연결리스트 자료구조인 list의 기능 중 push_back, pop_back을 구현하면서 연결리스트를 간단히 구현해보도록 하겠다. 여기서 노드를 삭제하거나, 삽입하는 기능을 추가하려면 연결리스트는 임의접근이 불가능하므로 iterator를 구현해야하는데 iterator까지 구현하면 볼륨이 너무 커질 것 같아서 생략하였다."
---

## 쓰레드 생성



#### CreateThread

```C++
HANDLE CreateThread(
  [in, optional]  LPSECURITY_ATTRIBUTES   lpThreadAttributes,
  [in]            SIZE_T                  dwStackSize,
  [in]            LPTHREAD_START_ROUTINE  lpStartAddress,
  [in, optional]  __drv_aliasesMem LPVOID lpParameter,
  [in]            DWORD                   dwCreationFlags,
  [out, optional] LPDWORD                 lpThreadId
)
```

- dwStatckSize
  - 스레드 스택의 크기
  - 프로세스가 시작되면 내부적으로 CreateThread 함수를 호출하여 프로세스의 주 스레드를 초기화한다.
  - 이때, CreateProcess는 실행 파일 내부에 저장되어 있는 값을 이용하여 dwStackSize의 매개변수 값을 결정한다.
  - 0을 인자로 넘겨주게되면 프로세스 기본크기로 할당괸다.
- lpStartAddress
  - 새로 생성되는 스레드가 호출할 스레드 함수의 주소.
- lpParameter
  - 스레드가 실행될 때 넘겨주는 매개변수 값
- dwCreationFlags
  - 0을 넘겨주면 바로 스케줄 가능 대상이 되고, `CREATE_SUSPENDED` 플래그를 사용하면, 스레드를 생성하고 초기화를 완료한 이후 SUSPEND상태로 있는다.
- lpThreadId
  - 쓰레드 고유의 아이디를 받을 포인터
  - 스레드 ID는 자기자신이 내부에서 사용하는 용도이고 외부에서 사용할일이 많이 없다.



#### _beginthreadex

_beginthreadex는 CreateThread와 같은 매개변수를 요구하지만, c런타임라이브러리를 위한 공간을 초기화하는 코드가 들어가있고, 자료형이 표준이로 바뀌어있다.



##### _beginthreadex 소스코드 분석

```c++
extern "C" uintptr_t __cdecl _beginthreadex(
    void*                    const security_descriptor,
    unsigned int             const stack_size,
    _beginthreadex_proc_type const procedure,
    void*                    const context,
    unsigned int             const creation_flags,
    unsigned int*            const thread_id_result
    )
{
    _VALIDATE_RETURN(procedure != nullptr, EINVAL, 0);
	
    // 우리가 지정한 paramaeter에 내가 지정한 Thread 시작지점 함수포인터를 합친다.
    unique_thread_parameter parameter(create_thread_parameter(procedure, context));
    if (!parameter)
    {
        return 0;
    }

    DWORD thread_id;
    HANDLE const thread_handle = CreateThread(
        reinterpret_cast<LPSECURITY_ATTRIBUTES>(security_descriptor),
        stack_size,
        thread_start<_beginthreadex_proc_type, true>, // thread_start라는 함수로 CreateThread를 호출한다.
        parameter.get(), // 시작점 함수 + 파라미터
        creation_flags,
        &thread_id);

    if (!thread_handle)
    {
        __acrt_errno_map_os_error(GetLastError());
        return 0;
    }

    if (thread_id_result)
    {
        *thread_id_result = thread_id;
    }

    // If we successfully created the thread, the thread now owns its parameter:
    parameter.detach();

    return reinterpret_cast<uintptr_t>(thread_handle);
}
```

- _beginthreadex에서 하는 일은 간단하다. 그냥 CreateThread를 하는데, 우리가 지정한 시작점 함수를 parameter앞에 붙힌다. 
- 그리고 c런타임에서 만든 thread_start라는 함수를 시작점으로 CreateThread를 실행하여 쓰레드를 만든다.

```c++
typedef struct __acrt_thread_parameter
{
    // 스레드 시작함수와 파라미터
    void*   _procedure; // 함수 시작점 주소 (우리가 지정한)
    void*   _context; // 함수 매개변수 (우리가 넘겨준)

    // 새로 생성된 스레드의 핸들. 오직 _beginthread로 했을 때만 초기화된다. (_beginthreadex)에선 안함
    // _beginthread로 생성된 스레드가 있다면, 이 핸들을 통해 반환한다.
    HANDLE _thread_handle;

    // 유저 스레드 프로시져에서 정의된 모듈의 핸들
    // 핸들을 얻을 수 없는 경우 null이다. 이 핸들을 사용하면 사용자 모듈의 참조 횟수를 증가시켜
    // 스레드가 실행되는 동안 모듈이 언로드되지 않도록 할 수 있다.
    // 스레드가 종료되면 이 핸들이 해제된다.
    HMODULE _module_handle;

    // MTA(다중 스레드 아파트)로 초기화하기 위해 스레드에서 RoInitialized가 호출된 경우 이 플래그는 true이다.
    bool    _initialized_apartment;
} __acrt_thread_parameter;

static __acrt_thread_parameter* __cdecl create_thread_parameter(
    void* const procedure,
    void* const context
    ) throw()
{
    unique_thread_parameter parameter(_calloc_crt_t(__acrt_thread_parameter, 1).detach());
    if (!parameter)
    {
        return nullptr;
    }

    parameter.get()->_procedure = procedure;
    parameter.get()->_context   = context;

    // 사용자 스레드 프로시저가 정의된 모듈의 카운트를 증가시켜, 스레드가 실행되는 동안 모듈이 계속 로드되도록 한다.
    // 스레드 프로시저가 반환되거나 _endthreadex가 호출되면 이 HMODULE을 해제한다.
    GetModuleHandleExW(
        GET_MODULE_HANDLE_EX_FLAG_FROM_ADDRESS,
        reinterpret_cast<LPCWSTR>(procedure),
        &parameter.get()->_module_handle);

    return parameter.detach();
}

```

- __acrt_thread_parameter라는 구조체를 만들어서 우리의 함수와 매개변수값을 하나의 구조체로 묶어 thread_start의 인자로 넘겨준다.



```c++
template <typename ThreadProcedure, bool Ex>
static unsigned long WINAPI thread_start(void* const parameter) throw()
{
    if (!parameter)
    {
        ExitThread(GetLastError());
    }

    __acrt_thread_parameter* const context = static_cast<__acrt_thread_parameter*>(parameter);
	
    // 새로운 스레드에서 사용할 ptd공간 동적 할당 및 초기화
    __acrt_getptd()->_beginthread_context = context;
	

    if (__acrt_get_begin_thread_init_policy() == begin_thread_init_policy_ro_initialize)
    {
        context->_initialized_apartment = __acrt_RoInitialize(RO_INIT_MULTITHREADED) == S_OK;
    }

    __try
    {
        ThreadProcedure const procedure = reinterpret_cast<ThreadProcedure>(context->_procedure);
        if constexpr (Ex)
        {
            // 우리의 시작점 함수 실행 및 함수가 종료되면 _endthreadex 호출
            _endthreadex(procedure(context->_context));
        }
        else
        {
            procedure(context->_context);
            _endthreadex(0);
        }
    }
    __except (_seh_filter_exe(GetExceptionCode(), GetExceptionInformation()))
    {
        // Execution should never reach here:
        _exit(GetExceptionCode());
    }

    // This return statement will never be reached.  All execution paths result
    // in the thread or process exiting.
    return 0;
}
```

- getptd() 함수를 호출했을 때, ptd가 없다면, ptd공간을 할당하고, 초기화하는 과정을 거친다. 그 이후 쓰레드의 함수와 매개변수를 ptd에 등록한다.
- 그 이후 우리가 정의한 함수를 실행한다.

#### ptd(Per-Thread Data)

- CRT에서 스레드마다 런타임함수가 실행될 떄 필요한 값을 저장해 놓기 위해 만들어놓은 구조체이다.

```C++
typedef struct __acrt_ptd
{
    // 시그널 핸들링과 런타임 에러를 도와주는 데이터 멤버 3종
    struct __crt_signal_action_t* _pxcptacttab;     // Pointer to the exception-action table
    EXCEPTION_POINTERS*           _tpxcptinfoptrs;  // Pointer to the exception info pointers
    int                           _tfpecode;        // Last floating point exception code

    terminate_handler  _terminate;  		// terminate() routine

    int                  _terrno;          // errno value
    unsigned long        _tdoserrno;       // _doserrno value

    unsigned int         _rand_state;      // Previous value of rand()

    // Per-thread strtok(), wcstok(), and mbstok() data:
    char*                _strtok_token;
    unsigned char*       _mbstok_token;
    wchar_t*             _wcstok_token;

    // Per-thread tmpnam() data:
    char*                _tmpnam_narrow_buffer;
    wchar_t*             _tmpnam_wide_buffer;

    // Per-thread time library data:
    char*                _asctime_buffer;  // Pointer to asctime() buffer
    wchar_t*             _wasctime_buffer; // Pointer to _wasctime() buffer
    struct tm*           _gmtime_buffer;   // Pointer to gmtime() structure

    char*                _cvtbuf;          // Pointer to the buffer used by ecvt() and fcvt().

    // Per-thread error message data:
    char*      _strerror_buffer;            // Pointer to strerror()  / _strerror()  buffer
    wchar_t*   _wcserror_buffer;            // Pointer to _wcserror() / __wcserror() buffer

    // Locale data:
    __crt_multibyte_data*                  _multibyte_info;
    __crt_locale_data*                     _locale_info;
    __crt_qualified_locale_data            _setloc_data;
    __crt_qualified_locale_data_downlevel* _setloc_downlevel_data;
    int                                    _own_locale;   // See _configthreadlocale() and __acrt_should_sync_with_global_locale()

    // The buffer used by _putch(), and the flag indicating whether the buffer
    // is currently in use or not.
    unsigned char  _putch_buffer[MB_LEN_MAX];
    unsigned short _putch_buffer_used;

    // The thread-local invalid parameter handler
    _invalid_parameter_handler _thread_local_iph;

    // 이 스레드가 _beginthread 또는 _beginthreadex에 의해 시작된 경우 이는 스레드가 생성된 컨텍스트를 가리킨다.
    // 이 스레드가 CRT에 의해 실행되지 않은 경우 null이다.
    __acrt_thread_parameter* _beginthread_context;

} __acrt_ptd;
```





### 쓰레드 종료



#### TerminateThread

- TerminateThread는 가장 극단적인 경우에만 사용해야하는 위험한 함수이다. 대상 스레드가 수행하는 작업을 정확하게 알고 있고, 종료 시 대상 스레드가 실행될 수 있는 모든 코드를 제어하는 경우에만 호출해야한다.
- 다음과 같은 문제가 발생할 수 있다.
  - 대상 스레드가 중요한 섹션을 소유하는 경우 중요한 섹션은 해제되지 않는다. (CriticalSection 말하는듯)
  - 대상 스레드가 힙에서 메모리를 할당하는 경우 힙 잠금이 해제되지 않는다.
  - 대상 스레드가 종료될 때 특정 kernel32 호출을 실행하는 경우 스레드 프로세스에 대한 kernel32 상태가 일치하지 않을 수 있다.
  - 대상 스레드가 공유 DLL의 전역 상태를 조작하는 경우 DLL의 상태가 삭제되어 DLL의 다른 사용자에게 영향을 줄 수 있다.
- 스레드를 종료한다고 해서 시스템에서 스레드 개체가 반드시 제거되는 것은 아이다. 마지막 스레드 핸들을 닫으면 스레드 개체가 삭제된다.



#### ExitThread

- C 코드에서 스레드를 종료하는 기본 방법이다. 그러나 c++코드에서는 소멸자를 호출하거나 다른 자동 정리를 수행하기 이전에 스레드가 종료된다. 따라서 C++에서는  스레드함수를 반황해야한다.
- 이 함수를 명시적으로 호출하거나 스레드 프로시저에서 return하면 현재 스레드 스택이 해제되고, 스레드가 종료된다. 연결된 모든 DLL의 진입점 함수는 스레드가 DLL에서 분리되고 있음을 나타내는 값으로 호출된다.
- CRT에 연결된 스레드는 _endthread를 호출해야한다. 이렇게 하지 않으면 스레드가 ExitThread를 호출할때 메모리 누수가 일어난다. (ptd가 정리되지 못한다.)



#### _endthreadex

- ptd와 CRT 자원을 해제 하고 ExitThread를 호출한다.
- 따라서 CRT쓰레드는 endthreadex를 호출해야한다



#### _endthreadex 코드분석

```c++
extern "C" void __cdecl _endthreadex(unsigned int const return_code)
{
    return common_end_thread(return_code);
}

static void __cdecl common_end_thread(unsigned int const return_code) throw()
{
    __acrt_ptd* const ptd = __acrt_getptd_noexit();
    //ptd 할당정보 없으면 바로 ExitThread호출
    if (!ptd)
    {
        ExitThread(return_code);
    }
    __acrt_thread_parameter* const parameter = ptd->_beginthread_context;
    //_beginthread_context 없으면 ExitThread호출
    if (!parameter)
    {
        ExitThread(return_code);
    }
    // 여기까지 _beginthreadex로 스레드를 생성하지 않았으면 발생하는 상황
    ----------------------------------------------------

    // RoInitialize가 호출된 경우 RoUninitialize 호출
    if (parameter->_initialized_apartment)
    {
        __acrt_RoUninitialize();
    }
	
    // thread 핸들 반환 (_beginthreadex로 생성했으면 넘어간다)
    if (parameter->_thread_handle != INVALID_HANDLE_VALUE && parameter->_thread_handle != nullptr)
    {
        CloseHandle(parameter->_thread_handle);
    }

    // DLL 참조카운트 1감소 후 ExitThread
    if (parameter->_module_handle != INVALID_HANDLE_VALUE && parameter->_module_handle != nullptr)
    {
        FreeLibraryAndExitThread(parameter->_module_handle, return_code);
    }
    else
    {
        ExitThread(return_code);
    }
}

```



#### 스레드의 종료 경우의 수

1. 스레드가 ExitThread 호출
   - 다른 스레드가 함수를 호출하는 스레드의 커널객체에 대한 핸들을 가지고 있지않는 경우에는 스레드 커널객체는 제거된다.
   - 가지고 있다면 모든 핸들을 반환하기 전까지 남아있다.
   - ptd가 정리되지 않는다.
2. 스레드가 _endthreadex 호출
   - ptd가 정리된다.
3. 스레드가 리턴
   - 지역변수에 대한 소멸자도 호출된다.