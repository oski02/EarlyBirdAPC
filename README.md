# Técnica EarlyBirdAPC empleando LowLevelAPI


Dando un vistazo al buen ejemplo de la Técnica EarlyBirdAPC con LowLevelAPI empleada en el post https://sevrosecurity.com/2020/04/13/process-injection-part-2-queueuserapc/ obervé que les faltaba poner una de las funciones en LowLevelAPI, asi que me puse manos a la obra e hice los cambios pertinentes para poder tener la técnica mostrada esta vez si a FULL LowLevelAPI. 

La funcion del post no migrada a LowLevelAPI era WaitForSingleObject de MediumLevelAPI y la pasamos a NtWaitForSingleObject para LowLevelAPI, de forma que podemos reducir las funciones importadas a Kernel32.dll del post mencionado y como quedan después al hacer el cambio.

# Probado en:

- System: Windows 11 (21H2 Build 22000.1696)
- IDE: Visual Studio 2022 

## Salida de dumpbin empleando el ejemplo del post sevrosesecurity: 

En el post deee sevrosesecurity cuando se hace uso del ejemplo de LowLevelAPI no se llega a emplear la funcion NtWaitForSingleObject y se usa la funcion WaitForSingleobject.

https://sevrosecurity.com/2020/04/13/process-injection-part-2-queueuserapc/

    KERNEL32.dll
             140003000 Import Address Table
             140003C58 Import Name Table
                     0 time date stamp
                     0 Index of first forwarder reference

                         610 WaitForSingleObject
                         27D GetLastError
                          F6 CreateProcessW
                         4FD RtlLookupFunctionEntry
                         504 RtlVirtualUnwind
                         5E6 UnhandledExceptionFilter
                         5A4 SetUnhandledExceptionFilter
                         232 GetCurrentProcess
                         5C4 TerminateProcess
                         3A8 IsProcessorFeaturePresent
                         470 QueryPerformanceCounter
                         233 GetCurrentProcessId
                         237 GetCurrentThreadId
                         30A GetSystemTimeAsFileTime
                         38A InitializeSListHead
                         3A0 IsDebuggerPresent
                         295 GetModuleHandleW
                         4F5 RtlCaptureContext




## Salida de dumpbin pasandola a Full LowLevelAPI:

Observamos que al hacer uso de NtWaitForSingleObject ya no nos aparece la importacion de la funcion WaitForSingleObject, reduciendo el numero de importaciones a kernel32.dll y siendo mas sigilosos frente a un EDR.

    KERNEL32.dll
             140003000 Import Address Table
             140003C58 Import Name Table
                     0 time date stamp
                     0 Index of first forwarder reference

                         27D GetLastError
                          F6 CreateProcessW
                         4FD RtlLookupFunctionEntry
                         504 RtlVirtualUnwind
                         5E6 UnhandledExceptionFilter
                         5A4 SetUnhandledExceptionFilter
                         232 GetCurrentProcess
                         5C4 TerminateProcess
                         3A8 IsProcessorFeaturePresent
                         470 QueryPerformanceCounter
                         233 GetCurrentProcessId
                         237 GetCurrentThreadId
                         30A GetSystemTimeAsFileTime
                         38A InitializeSListHead
                         3A0 IsDebuggerPresent
                         295 GetModuleHandleW
                         4F5 RtlCaptureContext


## Syswhispers 

Para poder generar el codigo ASM empleamos como en el ejemplo del post syswhispers pero añadiendo NtWaitForSingleObject, como vemos aqui:


`python syswhispers.py -f NtAllocateVirtualMemory,NtWriteVirtualMemory,NtCreateThreadEx,NtWaitForSingleObject,NtClose,NtCreateProcess,NtSuspendThread,NtQueueApcThread,NtResumeThread -a x64 -l masm --out-file syscalls
`


