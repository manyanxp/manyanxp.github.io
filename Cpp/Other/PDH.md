[UP](../index.md)

- **<font color="#006e54">Writing Performance Data to a Log File</font>**

https://msdn.microsoft.com/en-us/library/windows/desktop/aa373228(v=vs.85).aspx

```cpp
#include "stdafx.h"
#include <windows.h>
#include <stdio.h>
#include <pdh.h>
#include <pdhmsg.h>

#pragma comment(lib, "pdh.lib")

CONST PWSTR COUNTER_PATH = L"\\Processor(0)\\% Processor Time";
CONST ULONG SAMPLE_INTERVAL_MS = 1000;

void DisplayCommandLineHelp(void)
{
	wprintf(L"The command line must include a valid log file name.\n");
}

void wmain(int argc, WCHAR **argv)
{
	HQUERY hQuery = NULL;
	HLOG hLog = NULL;
	PDH_STATUS pdhStatus;
	DWORD dwLogType = PDH_LOG_TYPE_CSV;
	HCOUNTER hCounter;
	DWORD dwCount;

	if (argc != 2)
	{
		DisplayCommandLineHelp();
		goto cleanup;
	}

	// Open a query object.
	pdhStatus = PdhOpenQuery(NULL, 0, &hQuery);

	if (pdhStatus != ERROR_SUCCESS)
	{
		wprintf(L"PdhOpenQuery failed with 0x%x\n", pdhStatus);
		goto cleanup;
	}

	// Add one counter that will provide the data.
	pdhStatus = PdhAddCounter(hQuery,
		COUNTER_PATH,
		0,
		&hCounter);

	if (pdhStatus != ERROR_SUCCESS)
	{
		wprintf(L"PdhAddCounter failed with 0x%x\n", pdhStatus);
		goto cleanup;
	}

	// Open the log file for write access.
	pdhStatus = PdhOpenLog(argv[1],
		PDH_LOG_WRITE_ACCESS | PDH_LOG_CREATE_ALWAYS,
		&dwLogType,
		hQuery,
		0,
		NULL,
		&hLog);

	if (pdhStatus != ERROR_SUCCESS)
	{
		wprintf(L"PdhOpenLog failed with 0x%x\n", pdhStatus);
		goto cleanup;
	}

	// Write 10 records to the log file.
	for (dwCount = 0; dwCount < 10; dwCount++)
	{
		wprintf(L"Writing record %d\n", dwCount);

		pdhStatus = PdhUpdateLog(hLog, NULL);
		if (ERROR_SUCCESS != pdhStatus)
		{
			wprintf(L"PdhUpdateLog failed with 0x%x\n", pdhStatus);
			goto cleanup;
		}

		// Wait one second between samples for a counter update.
		Sleep(SAMPLE_INTERVAL_MS);
	}

cleanup:

	// Close the log file.
	if (hLog)
		PdhCloseLog(hLog, 0);

	// Close the query object.
	if (hQuery)
		PdhCloseQuery(hQuery);
}
```