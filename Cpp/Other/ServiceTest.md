[UP](../index.md)

# Service Application Sample.

### Logger Class
```cpp
#ifndef LOGGER_HEADER
#define LOGGER_HEADER

#include "stdafx.h"
#include <string>
#include <fstream>

class Logger
{
private:
	Logger() {}
	virtual ~Logger() {}

public:
	Logger(const Logger&) = delete;
	void operator=(const Logger&) = delete;

public:
	static Logger& getInstance()
	{
		static Logger instance;
		return instance;
	}

	void LogInfo(const std::string& mes)
	{
		std::ofstream log;
		log.open("D:\\test.txt", std::ios::app);
		if (!log.is_open())
			return;

		log << mes << "\n";

		log.close();
	}
};

#endif // !LOGGER_HEADER
```

### Sample Service Program.

```cpp
#include "stdafx.h"
#include <Windows.h>
#include "logger.hpp"

/*
ロガー
*/
Logger& g_logger = Logger::getInstance();

SERVICE_STATUS        g_ServiceStatus = { 0 };
SERVICE_STATUS_HANDLE g_StatusHandle = NULL;
HANDLE                g_ServiceStopEvent = INVALID_HANDLE_VALUE;

/*
*/
VOID WINAPI ServiceMain(DWORD argc, LPTSTR *argv);
VOID WINAPI ServiceCtrlHandler(DWORD);
DWORD WINAPI ServiceWorkerThread(LPVOID lpParam);

#define SERVICE_NAME  _T("My Sample Service")

VOID WINAPI ServiceMain(DWORD argc, LPTSTR *argv)
{
	DWORD Status = E_FAIL;

	g_StatusHandle = RegisterServiceCtrlHandler(SERVICE_NAME, ServiceCtrlHandler);

	if (g_StatusHandle == NULL)
	{
		g_logger.LogInfo("異常");
		return;
	}

	ZeroMemory(&g_ServiceStatus, sizeof(g_ServiceStatus));
	g_ServiceStatus.dwServiceType = SERVICE_WIN32_OWN_PROCESS;
	g_ServiceStatus.dwControlsAccepted = 0;
	g_ServiceStatus.dwCurrentState = SERVICE_START_PENDING;
	g_ServiceStatus.dwWin32ExitCode = 0;
	g_ServiceStatus.dwServiceSpecificExitCode = 0;
	g_ServiceStatus.dwCheckPoint = 0;

	if (SetServiceStatus(g_StatusHandle, &g_ServiceStatus) == FALSE)
	{
		g_logger.LogInfo(
			"My Sample Service: ServiceMain: SetServiceStatus returned error");
	}

	g_ServiceStopEvent = CreateEvent(NULL, TRUE, FALSE, NULL);
	if (g_ServiceStopEvent == NULL)
	{
		g_ServiceStatus.dwControlsAccepted = 0;
		g_ServiceStatus.dwCurrentState = SERVICE_STOPPED;
		g_ServiceStatus.dwWin32ExitCode = GetLastError();
		g_ServiceStatus.dwCheckPoint = 1;

		if (SetServiceStatus(g_StatusHandle, &g_ServiceStatus) == FALSE)
		{
			g_logger.LogInfo(
				"My Sample Service: ServiceMain: SetServiceStatus returned error");
		}
		return;
	}

	// Tell the service controller we are started
	g_ServiceStatus.dwControlsAccepted = SERVICE_ACCEPT_STOP;
	g_ServiceStatus.dwCurrentState = SERVICE_RUNNING;
	g_ServiceStatus.dwWin32ExitCode = 0;
	g_ServiceStatus.dwCheckPoint = 0;

	if (SetServiceStatus(g_StatusHandle, &g_ServiceStatus) == FALSE)
	{
		g_logger.LogInfo(
			"My Sample Service: ServiceMain: SetServiceStatus returned error");
	}

	HANDLE hThread = CreateThread(NULL, 0, ServiceWorkerThread, NULL, 0, NULL);

	WaitForSingleObject(hThread, INFINITE);


	CloseHandle(g_ServiceStopEvent);

	g_ServiceStatus.dwControlsAccepted = 0;
	g_ServiceStatus.dwCurrentState = SERVICE_STOPPED;
	g_ServiceStatus.dwWin32ExitCode = 0;
	g_ServiceStatus.dwCheckPoint = 3;

	if (SetServiceStatus(g_StatusHandle, &g_ServiceStatus) == FALSE)
	{
		g_logger.LogInfo(
			"My Sample Service: ServiceMain: SetServiceStatus returned error");
	}

	return;
}

VOID WINAPI ServiceCtrlHandler(DWORD CtrlCode)
{
	switch (CtrlCode)
	{
	case SERVICE_CONTROL_STOP:
		g_logger.LogInfo("Recived SERVICE_CONTROL_STOP");

		if (g_ServiceStatus.dwCurrentState != SERVICE_RUNNING)
			break;

		g_ServiceStatus.dwControlsAccepted = 0;
		g_ServiceStatus.dwCurrentState = SERVICE_STOP_PENDING;
		g_ServiceStatus.dwWin32ExitCode = 0;
		g_ServiceStatus.dwCheckPoint = 4;

		if (SetServiceStatus(g_StatusHandle, &g_ServiceStatus) == FALSE)
		{
			g_logger.LogInfo(
				"My Sample Service: ServiceCtrlHandler: SetServiceStatus returned error");
		}

		// This will signal the worker thread to start shutting down
		SetEvent(g_ServiceStopEvent);

		break;

	default:
		break;
	}
}

DWORD WINAPI ServiceWorkerThread(LPVOID lpParam)
{
	g_logger.LogInfo("Start Service Worker Thread.");

	while (WaitForSingleObject(g_ServiceStopEvent, 0) != WAIT_OBJECT_0)
	{

		Sleep(3000);
	}
	g_logger.LogInfo("End Service Worker Thread.");

	return ERROR_SUCCESS;
}

int main()
{
	g_logger.LogInfo("start.");
	g_logger.LogInfo("start.");

	SERVICE_TABLE_ENTRY ServiceTable[] =
	{
		{ SERVICE_NAME, (LPSERVICE_MAIN_FUNCTION)ServiceMain },
		{ NULL, NULL }
	};

	if (StartServiceCtrlDispatcher(ServiceTable) == FALSE)
	{
		auto code = GetLastError();
		g_logger.LogInfo("GetLastError:" + std::to_string(code));
		return code;
	}

    return 0;
}

```