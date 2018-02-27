[UP](../index.md)

## デイバイス情報関連

- **<font color="#006e54">Device Installation Reference in SDK header files</font>**

https://msdn.microsoft.com/en-us/library/windows/hardware/mt813623(v=vs.85).aspx

- **<font color="#006e54">SetupDiEnumDeviceInfo</font>**

https://msdn.microsoft.com/en-us/library/windows/hardware/ff551010(v=vs.85).aspx

- **<font color="#006e54">SetupDiGetDeviceRegistryProperty</font>**

https://msdn.microsoft.com/en-us/library/windows/hardware/ff551967(v=vs.85).aspx

## Sample1

```cpp
#include "stdafx.h"
#include <Windows.h>
#include <setupapi.h>
#include <iostream>
#include <string>
#pragma	comment(lib,"setupapi.lib")


int main()
{
	bool			ret;
	DWORD			dwIndex;
	TCHAR*			pszName;
	DWORD			dwSize;
	DWORD			dwRegType;
	BOOL			bRet;
	HDEVINFO		hDevInfo;
	SP_DEVINFO_DATA	sDevInfo;
	std::wstring		strMessage;

	hDevInfo = ::SetupDiGetClassDevs(NULL, 0, 0, DIGCF_PRESENT | DIGCF_ALLCLASSES);
	if (hDevInfo == INVALID_HANDLE_VALUE)
		return false;

	ret = false;
	dwIndex = 0;
	::ZeroMemory(&sDevInfo, sizeof(SP_DEVINFO_DATA));
	sDevInfo.cbSize = sizeof(SP_DEVINFO_DATA);
	while (true)
	{
		bRet = ::SetupDiEnumDeviceInfo(hDevInfo, dwIndex++, &sDevInfo);
		if (bRet == FALSE)
			break;

		ret = false;

		//デバイス名（説明）の取得
		dwSize = 0;
		bRet = ::SetupDiGetDeviceRegistryProperty(hDevInfo, &sDevInfo, SPDRP_DEVICEDESC, &dwRegType, NULL, 0, &dwSize);
		pszName = new TCHAR[dwSize];
		if (pszName == NULL)
			break;

		bRet = ::SetupDiGetDeviceRegistryProperty(hDevInfo, &sDevInfo, SPDRP_DEVICEDESC, &dwRegType, (BYTE*)pszName, dwSize, &dwSize);
		if (bRet == FALSE)
		{
			delete	pszName;
			continue;
		}

		strMessage += std::wstring(pszName);

		delete	pszName;

		{
			//フレンドリーネームの取得
			//取得できないことが多いので取得できない場合もcontinueしない
			dwSize = 0;
			bRet = ::SetupDiGetDeviceRegistryProperty(hDevInfo, &sDevInfo, SPDRP_FRIENDLYNAME, &dwRegType, NULL, 0, &dwSize);
			pszName = new TCHAR[dwSize];
			if (pszName)
			{
				bRet = ::SetupDiGetDeviceRegistryProperty(hDevInfo, &sDevInfo, SPDRP_FRIENDLYNAME, &dwRegType, (BYTE*)pszName, dwSize, &dwSize);

				if (bRet)
				{
					strMessage += std::wstring(L" (");
					strMessage += pszName;
					strMessage += std::wstring(L")");
				}

				delete	pszName;
			}

		}

		strMessage += std::wstring(L"\n");
		ret = true;
	}

	::SetupDiDestroyDeviceInfoList(hDevInfo);

	std::wcout.imbue(std::locale(""));
	std::wcout << strMessage << std::endl;

	return	ret;
}
```

## Sample2

```cpp
#include "stdafx.h"
#define STRICT
#include <Windows.h>
#include <setupapi.h>
#define INITGUID
#include <devpkey.h>
#undef INITGUID

#pragma comment(lib, "setupapi.lib")

// デバイスの文字列型プロパティをヒープに確保して返します。
LPTSTR HeapAllocDevicePropertyString(
	HANDLE              HeapHandle,
	HDEVINFO            DeviceInfoSet,
	PSP_DEVINFO_DATA    DeviceInfoData,
	const DEVPROPKEY*   PropertyKey,
	DEVPROPTYPE*        PropertyType,
	PDWORD              CopiedSize,
	DWORD               Flags)
{
	DEVPROPTYPE PropType;
	DWORD Size;
	if (!SetupDiGetDeviceProperty(
		DeviceInfoSet,
		DeviceInfoData,
		PropertyKey,
		&PropType,
		nullptr, 0,
		&Size,
		0) && GetLastError() != ERROR_INSUFFICIENT_BUFFER) {
		if (PropertyType != nullptr) {
			*PropertyType = DEVPROP_TYPE_NULL;
		}
		if (CopiedSize != nullptr) {
			*CopiedSize = 0;
		}
		return nullptr;
	}
	if (PropertyType != nullptr) {
		*PropertyType = PropType;
	}
	if (CopiedSize != nullptr) {
		*CopiedSize = Size;
	}
	if (PropType != DEVPROP_TYPE_STRING) {
		return nullptr;
	}
	LPTSTR buffer = (LPTSTR)HeapAlloc(HeapHandle, 0, Size);
	if (!SetupDiGetDeviceProperty(
		DeviceInfoSet,
		DeviceInfoData,
		PropertyKey,
		&PropType,
		(PBYTE)buffer, Size,
		&Size,
		0))
	{
		HeapFree(HeapHandle, 0, buffer);
		return nullptr;
	}
	return buffer;
}

#include <iostream>

#if defined(UNICODE) || defined(_UNICODE)
#define tcout std::wcout
#else
#define tcout std::cout
#endif

void main()
{
	// std::wcoutの日本語文字化けを回避
	std::wcout.imbue(std::locale("Japanese", std::locale::ctype));

	// 現在システムに存在する全てのクラス・インターフェイスを列挙
	tcout << TEXT("DIGCF_ALLCLASSES | DIGCF_PRESENT") << std::endl;
	HDEVINFO DevInfoHandle = SetupDiGetClassDevs(
		nullptr, nullptr, nullptr,
		DIGCF_ALLCLASSES | DIGCF_PRESENT);
	SP_DEVINFO_DATA DevInfoData = { sizeof SP_DEVINFO_DATA };
	for (DWORD i = 0; SetupDiEnumDeviceInfo(DevInfoHandle, i, &DevInfoData); i++)
	{
		LPTSTR ClassName = HeapAllocDevicePropertyString(
			GetProcessHeap(),
			DevInfoHandle,
			&DevInfoData,
			&DEVPKEY_Device_Class,
			nullptr,
			nullptr,
			0);
		LPTSTR DeviceDesc = HeapAllocDevicePropertyString(
			GetProcessHeap(),
			DevInfoHandle,
			&DevInfoData,
			&DEVPKEY_Device_DeviceDesc,
			nullptr,
			nullptr,
			0);
		tcout
			<< (ClassName ? ClassName : TEXT(""))
			<< TEXT(" - ")
			<< (DeviceDesc ? DeviceDesc : TEXT(""))
			<< std::endl;
		HeapFree(GetProcessHeap(), 0, DeviceDesc);
		HeapFree(GetProcessHeap(), 0, ClassName);
	}
	SetupDiDestroyDeviceInfoList(DevInfoHandle);
}
```