# OS#pragma comment(lib,"Ws2_32.lib")

#include <sys/types.h>
#include <WinSock2.h>
#include <Ws2tcpip.h>

#include <ctime>
#include <iostream>
#include <vector>
#include <string>
#include <math.h>
#define PORT "12345"

//функция для проверки на ошибки
void CheckForError(int result)
{

	if (result != 0)
	{
		printf("FALL!!! CheckForError result = %d", result);
		printf("\n");
		exit(0);
	}
}

class Client
{
private:
	int ID_thread;
	SOCKET Socket;
public:
	Client(SOCKET * in, int key) :Socket(*in), ID_thread(key) {}
	Client() {}
	SOCKET getsock() { return Socket; }
	int getID() { return ID_thread; }
	void changeID(int n) { ID_thread = n; }
};

//Сокет для подключения, хранения подключенных и для прослушки + подсчет количества подключенных 
SOCKET Connect;
SOCKET Listen;
int Count = 0; /*кол-во подключаемых пользователей, изначально 0*/

std::vector<Client> Connection(100);

//Удаляет отключенных пользователей
void decompres(int n)
{
	for (int i = n; i < Count - 1; i++)
	{
		Connection[i] = Connection[i + 1];
		Connection[i].changeID(Connection[i].getID() - 1);
	}
	Connection[Count].~Client();
}

//Функция для отправки-приема сообщений, принимает сообщение и рассылает всем подключенным пользователям
bool exit(char * out)
{
	int i = 0;
	char ex[] = "EXIT";
	char*temp = out;
	while (temp[0] != ':')
		temp++;
	temp++;
	temp++;
	if (strcmp(temp, ex) == 0)
		return true;
	return false;
}

void out_m(char * msg)
{
	char name[500];
	char * temp = msg;
	int i = 0;
	while (temp[0] != ':')
	{
		name[i++] = temp[0];
		temp++;
	}
	ZeroMemory(msg, sizeof(char) * 512);
	for (int j = 0; j < i; j++)
		msg[j] = name[j];
}

void SendM(int ID)
{
	setlocale(LC_ALL, "Russian");
	for (;; Sleep(75))
	{
		char msg[512];
		ZeroMemory(msg, sizeof(char) * 512);
		int iResult = recv(Connection[ID].getsock(), msg, 512, NULL);
		std::cout << msg << std::endl;
		if (exit(msg))
		{
			out_m(msg);
			strcat_s(msg, " покинул(а) чат");
			for (int i = 0; i <= Count; i++)
			{
				if (i == ID)continue;
				send(Connection[i].getsock(), msg, 512, NULL);
			}
			closesocket(Connection[ID].getsock());
			decompres(ID);
			Count--;
			break;
		}
		if (iResult>0)
		{
			for (int i = 0; i <= Count; i++)
			{
				if (i == ID)continue;
				send(Connection[i].getsock(), msg, 512, NULL);
			}
		}
	}
	printf("%d покинул(а) чат\n", ID);
}


//выясняет, отлаживается ли вызывающий процесс (динамическая защита)
void fastcall(int p)
{
	bool check1 = IsDebuggerPresent();
	if (check1) {
		printf("Меня отлаживают...\n");
		exit(0);
	}
	//else
	//printf("Нет отладчика...\n");
}

int fluse(std::string pass) {
	std::string parol = "woody";
	if (pass != parol)
		exit(0);
}

int main()
{
	//для защиты ключом просим ввести ключ
	setlocale(LC_ALL, "Russian");
	char Key[20];
	printf("Введите ключ: ");
	std::cin >> Key;
	fluse(Key);


	printf("Добро пожаловать!\n");
	WSAData  ws;
	WORD version = MAKEWORD(2, 2);
	int MasterSocket = WSAStartup(version, &ws);
	if (MasterSocket != 0)
	{
		return 0;
	}
	struct addrinfo hints;
	struct addrinfo * result;
	ZeroMemory(&hints, sizeof(hints));


	//Задание сокетов
	hints.ai_family = AF_INET;
	hints.ai_flags = AI_PASSIVE;
	hints.ai_socktype = SOCK_STREAM;
	hints.ai_protocol = IPPROTO_TCP;

	//Установка ip и порта
	//printf("Введите ip-адрес: ");
	//std::string iport; std::cin >> iport;
	//printf("Введите port : ");
	//std::string port; std::cin >> port;
	//getaddrinfo(iport.c_str(), port.c_str(), &hints, &result);
	getaddrinfo(NULL, PORT, &hints, &result);

	//Заполнение сокета listen
	fastcall(Listen = socket(result->ai_family, result->ai_socktype, result->ai_protocol));//дин
	fastcall(bind(Listen, result->ai_addr, result->ai_addrlen));//дин
	fastcall(listen(Listen, SOMAXCONN));//дин
	freeaddrinfo(result);

	//Начало работы сервера
	std::cout << "\nЖдём подключений..." << std::endl;
	char c_connect[] = "*Клиент №";
	while (1)
	{
		//проверка на получение сигнала от кого-нибудь
		if (Connect = accept(Listen, NULL, NULL))
		{
			std::cout << c_connect << ' ' << Count << std::endl;
			Connection[Count] = Client::Client(&Connect, Count);
			Count++;
			CreateThread(NULL, NULL, (LPTHREAD_START_ROUTINE)SendM, (LPVOID)(Count - 1), NULL, NULL);
		}
	}
	return 0;
}
