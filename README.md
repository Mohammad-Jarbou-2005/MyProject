# MyProject
#include<iostream>
#include<string>
#include<ctime>
#include<cstdlib>
#include <iomanip>
#include<vector>
#include<fstream>
#include<iomanip>

using namespace std;

const string ClientsFileName = { "Clients.txt" };
const string UsersFileName = { "Users" };

enum enClientChoise {
	showClientLit = 1, AddNewClient = 2,
	DeleteClient = 3, UpdateClient = 4,
	FindCleint = 5, Transactions = 6,
	ManageUsers = 7, Logout = 8
};

enum enTransactions { Deposit = 1, Withdraw = 2, TotalBalance = 3, MainMenue = 4 };

enum enUsersChoise {
	ListUsers=1,AddNewuser=2,
	DeleteUser=3,Updateuser=4,
	FindUser=5,Mainmenue=6
};

struct stClient {
	string Name;
	string Phone;
	string pinCode;
	string AccountNumber;
	double AccountBalance;
};

struct stUsers {
	string Name;
	string Password;
	int permissions;
};

void UpdateClientByAccountNumber(vector<stClient> vClients);

void ShowLoginScreen();

void StartManageUsers(stUsers);

vector<stUsers> FormFileUsersToVector(string FileName);

vector<stClient>LoadDataFormFile(string FileName);

void  ChooseManageUsers(int numChoise,vector<stUsers> vUsers,stUsers);

string ReadString(string Massage) {

	cout << Massage <<"? ";
	string S;
	
	getline(cin>>ws,S);

	return S;

}

vector<string> SplitString(string S, string Demil) {

	string Word;
	vector <string > vWords;
	int pos = 0;

	while ((pos = S.find(Demil)) != std::string::npos)
	{
		Word = S.substr(0, pos);

		if (Word != "") 

			vWords.push_back(Word);
		

		S.erase(0, pos+Demil.length());
	}

	if (S != "") 

		vWords.push_back(S);
	

	return vWords;
}

string LowerString(string S)
{
	for (int i = 0; i < S.length(); i++) {
		S[i] = tolower(S[i]);
	}
	return S;
}

string FromVectorToString( string Demil,vector <string> vWords)
{

	string S1="";

	for (string &n : vWords)
	{
			S1 += n + Demil;
		
	}
	S1 = S1.substr(0, S1.length() - Demil.length());

	return S1;
}

bool ChackCAccountNumber(string AccountNumber)
{
	vector <stClient> vClients = LoadDataFormFile(ClientsFileName);
	stClient Client;
	for (stClient& C : vClients)
	{
		if (AccountNumber == C.AccountNumber)
		{
			Client = C;
			return true;
		}
	}
	return false;
}

stClient ReadNewClient() {
	
	stClient Client;
	cout << "Enter Account Number : ";
	bool Chack;

	do
	{
		getline(cin >>ws,Client.AccountNumber);
		Chack = ChackCAccountNumber(Client.AccountNumber);

		if (Chack)
		{
			cout << "Client with [" << Client.AccountNumber << "already exists ,Enter another Account Number";
		}

	} while (Chack);

	cout << "Enter PinCode : ";
	getline(cin, Client.pinCode);

	cout << "Enter Name : ";
	getline(cin, Client.Name);

	cout << "Enter Phone : ";
	getline(cin, Client.Phone);

	cout << "Enter Account Balance : ";
	cin >> Client.AccountBalance;
	
	return Client;
}

stClient FromLineToStruct(string S, string Demil) {

	short pos = S.find(Demil);
	stClient Client;

	Client.AccountNumber = S.substr(0, pos);
	S = S.erase(0, pos + Demil.length());

	pos = S.find(Demil);
	Client.pinCode = S.substr(0, pos);
	S = S.erase(0, pos + Demil.length());

	pos = S.find(Demil);
	Client.Name = S.substr(0, pos);
	S = S.erase(0, pos + Demil.length());

	pos = S.find(Demil);
	Client.Phone = S.substr(0, pos);
	S.erase(0, pos + Demil.length());

	Client.AccountBalance = stof(S);

	return Client;

}

string FromStructClientToLine(stClient Client)
{
	string S;
	string Demil = "#//#";

	S = Client.AccountNumber+Demil;
	S += Client.pinCode+Demil;
	S += Client.Name + Demil;
	S += Client.Phone + Demil;
	S += to_string(Client.AccountBalance);

	return S;

}

void PrintStructTypeClient(stClient Client)
{
	cout << "The following are the client details : \n";
	cout << "\n------------------------------------------\n\n";

	cout << "Account Number  : " << Client.AccountNumber << endl;
	cout << "Pin Code        : " << Client.pinCode << endl;
	cout << "Name            : " << Client.Name << endl;
	cout << "Phone           : " << Client.Phone << endl;
	cout << "Account Balance : " << Client.AccountBalance << endl;

	cout << "\n------------------------------------------\n";

}

void AddDataLieToFile(string FileName, string LineData)
{
	fstream MyFile;
	MyFile.open(FileName, ios::out | ios::app);

	if (MyFile.is_open())
	{

		MyFile << LineData << endl;

		MyFile.close();
	}
}

void AddingNewClient()
{

	stClient Client = ReadNewClient();
	string Line = FromStructClientToLine(Client);
	AddDataLieToFile(ClientsFileName, Line);
	
}

void SaveClients(vector<stClient> vClients, string FileName)
{
	fstream MyFile;
	string Line;

	MyFile.open(FileName, ios::out);


	if (MyFile.is_open())
	{
		for (stClient& n : vClients)
		{
				Line = FromStructClientToLine(n);
				MyFile << Line << endl;
			
		}
		MyFile.close();
	}
	

}

void AddClients()
{
	char SaveAgane = 'y';

	do {
		system("cls");

		cout << "----------------------------------\n";
		cout << "Add New Clients Screen \n";
		cout << "----------------------------------\n";

		cout << "Adding New Client : \n\n";

		AddingNewClient();
		cout << "Client Added Successfully , do you want to add more Clients? Y/N? ";
		cin >> SaveAgane;

	} while (toupper(SaveAgane) == 'Y');
}

stClient ConvertClientRecord(string Line)
{
	vector<string> vClient;
	stClient Client;

	vClient = SplitString(Line, "#//#");

	Client.AccountNumber = vClient[0];
	Client.pinCode = vClient[1];
	Client.Name = vClient[2];
	Client.Phone = vClient[3];
	Client.AccountBalance = stod(vClient[4]);

	return Client;
}

vector<stClient>LoadDataFormFile(string FileName)
{
	fstream MyFile;
	vector<stClient> vClients;


	MyFile.open(FileName, ios::in);

	if (MyFile.is_open()) {

		string Line;
		stClient Client;


		while (getline(MyFile, Line))
		{

			Client = ConvertClientRecord(Line);
			vClients.push_back(Client);

		}
	}
	return vClients;
}

void ShowClient(stClient Client)
{
	cout << "| " << setw(15) << left << Client.AccountNumber;
	cout << "| " << setw(12) << left << Client.pinCode;
	cout << "| " << setw(40) << left << Client.Name;
	cout << "| " << setw(15) << left << Client.Phone;
	cout << "| " << setw(15) << left << Client.AccountBalance;

}

void ShowAllClients(vector <stClient> vClients)
{
	cout << "\t\t\t\t Client Line (" << vClients.size() << ") Client(s)\n";
	cout << "-------------------------------------------------------------------------------------------------------------------\n";
	cout << "| " << left << setw(15) << "Account Number";
	cout << "| " << left << setw(12) << "Pin Code ";
	cout << "| " << left << setw(40) << "Client Name ";
	cout << "| " << left << setw(15) << "phone ";
	cout << "| " << left << setw(15) << "Balance";
	cout << endl;
	cout << "\n-------------------------------------------------------------------------------------------------------------------\n\n";


	for(stClient &c: vClients)
	{
		ShowClient(c);
		cout << endl;
	}
	cout << "\n\n-------------------------------------------------------------------------------------------------------------------\n";


}

bool ChackFindClientByAccountNumber(vector<stClient> vClients, string AccountNumber)
{
	for (stClient& C : vClients)
	{
		if (C.AccountNumber == AccountNumber)
		{
			return true;
		}
	}

	cout << "Client With Account Number (" << AccountNumber << ") is not Found \n";
	return false;
}

void DeletedClientByAccountNummber(vector<stClient> vClients) {
	vector<stClient> vNewClients;
	char Delete;
	string AccountNumber;

	cout << "---------------------------------\n";
	cout << "\tDelete Client Screen\n";
	cout << "---------------------------------\n";

	cin.ignore();
	AccountNumber = ReadString("Please Entre Account Number To Client Delete : ");
	
	if (ChackFindClientByAccountNumber(vClients,AccountNumber))
	{
		for (stClient& n : vClients) {
			if (n.AccountNumber == AccountNumber)
			{
				PrintStructTypeClient(n);
				cout << "Do You sure You Want Delete ? Y/N? ";
				cin >> Delete;

				if (tolower(Delete) == 'y');

				else
				{
					return;
				}
			}
			else
			{
				vNewClients.push_back(n);
			}
		}

		SaveClients(vNewClients, ClientsFileName);

	}

	else
		return;

}

void FindClientByAccountNumber(vector<stClient> vClients) {
	string AccountNumber;

	cin.ignore();
	AccountNumber = ReadString("Please Entre Account Number To Client Find : ");

	for (stClient& C : vClients)
	{
		if (C.AccountNumber == AccountNumber) {
			PrintStructTypeClient(C);
				return ;
		}
	}

	cout << "Client With Account Number (" << AccountNumber << ") is not Found \n";
	
}

stClient UpdateStructTypeClient(string AccountNumber)
{
	stClient Client;

	Client.AccountNumber = AccountNumber;
	cout << "Enter Pin Code : ";
	getline(cin, Client.pinCode);

	cout << "Enter Name : ";
	getline(cin, Client.Name);

	cout << "Enter Phone : ";
	getline(cin, Client.Phone);

	cout << "Enter Balance : ";
	cin >> Client.AccountBalance;

	return Client;
}

int ReadNumber(string Massage)
{
	int Number;
	cout << Massage;
	cin >> Number;
	
	return Number;
}

void UpdateClientByAccountNumber(vector<stClient> vClients)
{
	
	char an = 'n';
	string AccountNumber;

	cout << "------------------------------------\n";
	cout << "\tUpdate Client Screen \n";
	cout << "------------------------------------\n";

	cin.ignore();
	AccountNumber = ReadString("Please Entre Account Number To Client Find : ");
	if (ChackFindClientByAccountNumber(vClients, AccountNumber))
	{
		for (stClient& n : vClients)
		{
			if (n.AccountNumber == AccountNumber)
			{
				PrintStructTypeClient(n);
				cout << "Do You Wnat Update ?Y/N? ";
				cin >> an;
				cin.ignore();
				if (tolower(an) == 'y')
				{
					n = UpdateStructTypeClient(AccountNumber);
					break;
				}
				else
					return;
			}

		}
		SaveClients(vClients, ClientsFileName);
	}

	else
		return;
}

void ShowMainMenue()
{
	cout << "============================================\n";
	cout << "\t\tMain Menue Screen                           \n";
	cout << "============================================\n";

	cout << "\t[1] Show Client List .\n";
	cout << "\t[2] Add New Client .\n";
	cout << "\t[3] Delete Client .\n";
	cout << "\t[4] Update Client Info .\n";
	cout << "\t[5] Find Client .\n";
	cout << "\t[6] Transactions .\n";
	cout << "\t[7] Mansge Users .\n";
	cout << "\t[8] Logout .\n";

	cout << "\n============================================\n";


}

void ShowTransactionsMenue()
{
	cout << "============================================\n";
	cout << "\t\tTransactions Meune Screen \n";
	cout << "============================================\n";

	cout << "\t[1] Deposit .\n";
	cout << "\t[2] Withdraw .\n";
	cout << "\t[3] Total Balance .\n";
	cout << "\t[4] Main Menue .\n";
	cout << "\n============================================\n";


}

void DepositBalance(vector<stClient> &vClients)
{
	cout << "-------------------------------------\n";
	cout << "\tDeposit Screen \n";
	cout << "-------------------------------------\n";


	string AccountNumber;
	bool Chack;
	cin.ignore();
	do {
		AccountNumber = ReadString("Please enter Account Number :");
		Chack = ChackCAccountNumber(AccountNumber);

		if (!Chack)
		{
			cout << "Client with [" << AccountNumber << "] does notexist . \n";
		}
	} while (!Chack);

	char sure;

	for (stClient &C : vClients)
	{
		
		if (C.AccountNumber == AccountNumber)
		{
			PrintStructTypeClient(C);
			int Bonse = ReadNumber("Please enter deposit amount : ");

			cout << "Are you sure you want preform this transaction ? Y/N? ";
			cin >> sure;
			if (tolower(sure) == 'y')
			{
				C.AccountBalance = C.AccountBalance + Bonse;

				cout << "\n\nDone sucssfully now Balance " << C.AccountBalance << endl;
				return;
			}
			else
				return;
		}
	}
}

void WithdrawBalance(vector <stClient> &vClients)
{
	cout << "-------------------------------------\n";
	cout << "\tWithdraw Screen \n";
	cout << "-------------------------------------\n";

	string AccountNumber;
	bool Chack;
	cin.ignore();

	do {
		AccountNumber = ReadString("Please enter Account Number :");
		Chack = ChackCAccountNumber(AccountNumber);

		if (!Chack)
		{
			cout << "Client with [" << AccountNumber << "] does notexist . \n";
		}
	} while (!Chack);

	char sure;

	for (stClient& C : vClients)
	{
		if (C.AccountNumber == AccountNumber)
		{
			int WithDraw;
			PrintStructTypeClient(C);
			do {
				
				WithDraw = ReadNumber("Please enter Withdraw amount ?");

				if (WithDraw > C.AccountBalance)
					cout << "Amount Exceeds the balance ,you can withdraw up to :" << C.AccountBalance << endl;

			} while (WithDraw > C.AccountBalance);

			cout << "Are you sure you want prtform this transactions ? y/n";
			cin >> sure;

			if (tolower(sure) == 'y')
			{
				C.AccountBalance -= WithDraw;
				cout << "\n\nDone sucssfully now Balance " << C.AccountBalance << endl;
				return;
			}

			else
				return;
		}
	}

}

void ShowBalanceScreen(vector<stClient> vClients)
{
	cout << "\t\t\t\t Balance Line (" << vClients.size() << ") Client(s)\n";
	cout << "------------------------------------------------------------------------------------\n\n";

	cout << left << setw(15) << "| Account Number ";
	cout << left << setw(40) << "| Client Name";
	cout << left << setw(15) << "| Balance " << endl;

	cout << "\n------------------------------------------------------------------------------------\n\n";

	cout << endl;
	for (stClient& C : vClients)
	{
		cout << "| " << left << setw(15) << C.AccountNumber;
		cout << "| " << left << setw(40) << C.Name;
		cout << "| " << left << setw(15) << C.AccountBalance;
		cout << endl;
	}

	cout << "\n------------------------------------------------------------------------------------\n\n";

}

void TotalBalanceOFAllClients(vector <stClient> vClients)
{
	int Totalbalance = 0;
	ShowBalanceScreen(vClients);

	for (stClient& C : vClients)
	{
		Totalbalance += C.AccountBalance;
	}

	cout << "\t\t\t\tTotal Balance = " << Totalbalance << endl;
}

void ChooseTransactions(enTransactions Option,vector<stClient> vClients)
{

	
	switch (Option) {

	case enTransactions::Deposit:
		system("cls");
		DepositBalance(vClients);
		SaveClients(vClients, ClientsFileName);
		return;

	case enTransactions::Withdraw:
		system("cls");
		WithdrawBalance(vClients);
		SaveClients(vClients, ClientsFileName);
		return;

	case enTransactions::TotalBalance:
		system("cls");
		TotalBalanceOFAllClients(vClients);
		return;
		
	}
}

void TransactionsOnBalance(vector <stClient> vClients)
{
	
	short Choose;
	do {
		system("cls");
		ShowTransactionsMenue();

		Choose = ReadNumber("Choose what do you want to do ? [1 to 4] ?");

		if (Choose == 4)
			return;

		ChooseTransactions(enTransactions(Choose), vClients);

		cout << "Press any key to go back to Main Menue...";

		system("pause>0");


	} while (true);
}

void ShowMansgeUsers()
{
	cout << "=====================================================\n";
	cout << "\t\tManage Users Menue Screen \n";
	cout << "=====================================================\n";
	cout << "\t[1] List Users .\n";
	cout << "\t[2] Add New Users .\n";
	cout << "\t[3] Delete Users .\n";
	cout << "\t[4] Update Users .\n";
	cout << "\t[5] Find Usres .\n";
	cout << "\t[6] Main Menue. \n";
}

bool ChackUsersNameIsFound(string Name)
{
	vector <stUsers> vUsers = FormFileUsersToVector(UsersFileName);
	for (stUsers& n : vUsers) {
		if (Name == n.Name)
			return true;
	}
	return false;
}

int MenuePermissions()
{
	int per = 0;
	char qua = 'y';

	cout << "Do you want to give access to : \n\n";
	
	cout << "Show Clients List ? y/n?";
	cin >> qua;
	if (tolower(qua) == 'y')
		per +=pow(2, int(enClientChoise::showClientLit)-1);

	cout << "Add New Client? y/n?";
	cin >> qua;
	if (tolower(qua) == 'y')
		per += pow(2, int(enClientChoise::AddNewClient) - 1);

	cout << "Delete Client ? y/n?";
	cin >> qua;
	if (tolower(qua) == 'y')
		per += pow(2, int(enClientChoise::DeleteClient) - 1);

	cout << "Update Client ? y/n?";
	cin >> qua;
	if (tolower(qua) == 'y')
		per += pow(2, int(enClientChoise::UpdateClient) - 1);

	cout << "Find Client ?y/n?";
	cin >> qua;
	if (tolower(qua) == 'y')
		per += pow(2, int(enClientChoise::FindCleint) - 1);

	cout << "Transaction ? y/n?";
	cin >> qua;
	if (tolower(qua) == 'y')
		per += pow(2, int(enClientChoise::Transactions) - 1);

	cout << "Manage Usres? y/n?";
	cin >> qua;
	if (tolower(qua) == 'y')
		per += pow(2, int(enClientChoise::ManageUsers) - 1);

	return per;
}

int PermissionsUser()
{
	char FullAccess;
	cout << "Do you want give Full access? Y/N?";
	cin >> FullAccess;

	if (tolower(FullAccess) == 'y')
		return -1;

	else
		return MenuePermissions();
}

stUsers ReadNewUser()
{
	stUsers User;
	string Name;
	do {
		 Name = ReadString("Enter UserName");
		if (ChackUsersNameIsFound(Name))
			cout << "User [" << Name << "] already exists ,Enter another UserName? \n";
		else
			break;

	} while (true);
	
		string Password = ReadString("Enter Password");

		User.Name = Name;
		User.Password = Password;
		User.permissions = PermissionsUser();

		return User;
}

string FormStrctToStringUsers(stUsers User)
{
	string Line = "";
	string Demil = "#//#";

	Line = User.Name + Demil;
	Line += User.Password + Demil;
	Line += to_string(User.permissions);

	return Line;
}

void AddNewUser()
{
	fstream Users;
	Users.open(UsersFileName, ios::out| ios::app);

	stUsers User = ReadNewUser();
	string Line = FormStrctToStringUsers(User);

	if (Users.is_open())
	{
		
		Users << Line << endl;
		Users.close();
	}
}

void AddUsers()
{
	char Addagain = 'y';
	do {
		system("cls");
		cout << "--------------------------------------------------------\n";
		cout << "\tAdd New User Screen \n";
		cout << "--------------------------------------------------------\n";
		cout << "Adding New User : \n\n";
		AddNewUser();

		cout << "User Added succefully, do you want to add more Users? Y/N?";
		cin >> Addagain;


	} while (tolower(Addagain) == 'y');
	cout << "press key to go back to Transction ...";
	system("pause>0");
}

int ChackFindUser(vector<stUsers> Users, string Name)
{
	int numberUser = 0;
	for (stUsers& n : Users)
	{
		numberUser++;
		if (n.Name == Name)
			return numberUser;
	}
	return 0;
}

void ResultresarchFindUser(vector<stUsers> Users)
{
	string Name = ReadString("Enter User Name ");
	int numberUser = ChackFindUser(Users, Name) ;

	if (numberUser)
	{
		cout << "The Information User : ";
		cout << "\n---------------------------------------------------\n";
		cout << "User Name   : " << Users[numberUser-1].Name << endl;
		cout << "Password    : " << Users[numberUser-1].Password << endl;
		cout << "Permissions : " << Users[numberUser-1].permissions << endl;
		cout << "---------------------------------------------------\n";

	}
	else
		cout << "User with UserName (" << Name << ")is not Found \n\n";

	cout << "press key to go back to Transction ...";
	system("pause>0");
}

void ShowInfoUser(stUsers User)
{
	cout << "| " << setw(20) << left << User.Name
		<< "| " << setw(10) << left << User.Password
		<< "| " << setw(10) << left << User.permissions << endl;
}

void ShowListUsers(vector<stUsers> vUsers)
{
	cout << "\n\t\t\t\tUsers List (" << vUsers.size() << ") " << "Users (s)\n";	
	cout << "\n ---------------------------------------------------------------------------------------------------------\n";
	cout << "| " << setw(20) << left << "User Name " << "| " << setw(10) << left << "Password" << "| " << setw(10) << left << "Permissions" << endl;
	cout << " ---------------------------------------------------------------------------------------------------------\n";

	for(stUsers &n :vUsers)
	{
		ShowInfoUser(n);
		cout << " ---------------------------------------------------------------------------------------------------------\n";

	}
	cout << "\n\nPerss any to go back Transaction Menue....\n";
	system("pause>0");
}

void ChooseUserofMainMenue(int Choose,vector<stClient> vClients,stUsers User)
{
	int p = (pow(2, Choose - 1));
	if ((p & User.permissions)|| User.permissions==-1||Choose==8)
	{
		switch (Choose)
		{

		case enClientChoise::showClientLit:
			system("cls");
			ShowAllClients(vClients);
			return;


		case enClientChoise::AddNewClient:
			AddClients();
			return;


		case enClientChoise::DeleteClient:
			system("cls");
			DeletedClientByAccountNummber(vClients);
			return;


		case enClientChoise::UpdateClient:
			system("cls");
			UpdateClientByAccountNumber(vClients);
			return;


		case enClientChoise::FindCleint:
			system("cls");
			FindClientByAccountNumber(vClients);
			return;

		case enClientChoise::Transactions:
			system("cls");
			TransactionsOnBalance(vClients);
			return;

		case enClientChoise::ManageUsers:
			system("cls");
			StartManageUsers(User);
			return;

		case enClientChoise::Logout:
			system("cls");
			ShowLoginScreen();
			return;
		}
	}
	else
	{
		system("cls");
		cout << "-----------------------------------------------\n";
		cout << "Access Denied,\n";
		cout << "You Don't Have Permissions To Do This \n Please Conact Your Admin .\n";
		cout << "-----------------------------------------------\n";

		return;
	}
}

short ReadMainMenueOption()
{
	
	short Number;
	cout << "Choose what do you want to do ? [1 to 8] ?";
	cin >> Number;

	return Number;
}


void StartSystemClients(stUsers User)
{
	vector<stClient> vClients;
	
	short Number;

	do {
		system("cls");

		vClients = LoadDataFormFile(ClientsFileName);
		

		ShowMainMenue();

		 Number = ReadMainMenueOption();

		ChooseUserofMainMenue((Number), vClients,User);


		cout << "Press any key to go back to Main Menue...";

		system("pause>0");

	} while (true);

}

stUsers ConvertUserRecord(string Line)
{
	vector <string> vUser = SplitString(Line, "#//#");
	stUsers User;
	User.Name = vUser[0];
	User.Password = vUser[1];
	User.permissions = stoi(vUser[2]);

	return User;
}

vector<stUsers> FormFileUsersToVector(string FileName)
{
	fstream Users;
	vector<stUsers> vUsers;

	Users.open(FileName, ios::in);

	if (Users.is_open())
	{
		string Line; 
		stUsers User; 

		while (getline(Users , Line))
		{
			User = ConvertUserRecord(Line);
			vUsers.push_back(User);

		}
	}
	return vUsers;
}

void HeadDeleteUser()
{
	cout << "------------------------------------------\n";
	cout << "\tDelete Users Screen \n";
	cout << "------------------------------------------\n";

}

void InfoUser(stUsers User)
{
	cout << "The Information User : ";
	cout << "\n---------------------------------------------------\n";
	cout << "User Name   : " << User.Name << endl;
	cout << "Password    : " << User.Password << endl;
	cout << "Permissions : " << User.permissions << endl;
	cout << "---------------------------------------------------\n";

}

void Erase(vector<stUsers> &Users,int index)
{
	for (int i = index; i < Users.size(); i++)
	{
		Users[i] = Users[i + 1];
	}
	Users.resize(Users.size() - 1);
}

void SaveUsersFile(vector<stUsers>Users)
{
	fstream myfile;
	myfile.open("Users", ios::out);
	if (myfile.is_open())
	{
		string line;
		for (stUsers& u : Users)
		{
			line = FormStrctToStringUsers(u);
			myfile << line << endl;
		}
		myfile.close();
	}
}

void HeadUpdate()
{
	cout << "---------------------------------------------\n";
	cout << "\tUpdate Users Screen \n";
	cout << "---------------------------------------------\n\n";

}

void UpdateUser(vector<stUsers> Users)
{
	HeadUpdate();

	string Name = ReadString("Enter Users Name");
	int numUser = ChackFindUser(Users, Name);
	char que;

	if (numUser)
	{
		InfoUser(Users[numUser - 1]);
		cout << "Are you sure you Update this User? y/n? ";
		cin >> que;
		if (tolower(que) == 'y')
		{
			string Password = ReadString("Enter Password ");
			Users[numUser].Password = Password;
			if (Name == "Admin");
			else
			Users[numUser - 1] .permissions= PermissionsUser();
		}
		SaveUsersFile(Users);
		cout << "press key to go back to Transction ...";
		system("pause>0");
	}

}

void DeleteUsers(vector<stUsers> Users)
{
	HeadDeleteUser();
	string Name = ReadString("Enter User Name ");
	int numUser = ChackFindUser(Users, Name);
	char que;

	if (numUser)
	{
		InfoUser(Users[numUser - 1]);
		cout << "Are you sure you Delete User? y/n? ";
		cin >> que;
		if (tolower(que) == 'y')
		{
			Erase(Users, numUser - 1);
			SaveUsersFile(Users);
			cout << "Deleted is sccessfully \n\n";
			cout << "press key to go back to Transction ...";
			system("pause>0");
		}
		else
			exit(1);
	}
}

void ScreenChooseManageUsers()
{
	cout << "===========================================================\n";
	cout << "\t\t\tManage Users Menuen Screen \n";
	cout << "===========================================================\n";
	cout << "\t[1] List Users .\n";
	cout << "\t[2] Add New User .\n";
	cout << "\t[3] Delete User .\n";
	cout << "\t[4] Update User .\n";
	cout << "\t[5] Find User .\n";
	cout << "\t[6] Main Menue .\n";
	cout << "===========================================================\n";

}

int ReadNumberChoiseManageMenueUsers()
{
	int numChoise;
	cout << "Choose want do you to do ? [ 1 to 6 ]? ";
	cin >> numChoise;

	return numChoise;

}

void StartManageUsers(stUsers User)
{
	int numChoise;
	vector<stUsers> vUsers;

	do {
		system("cls");
		ScreenChooseManageUsers();

		vUsers = FormFileUsersToVector(UsersFileName);

		 numChoise = ReadNumberChoiseManageMenueUsers();
		 
		 ChooseManageUsers(numChoise, vUsers,User);


	} while (true);
}

void ChooseManageUsers(int numChoise, vector<stUsers> vUsers,stUsers User)
{

	switch (enUsersChoise(numChoise))
	{
	case enUsersChoise::ListUsers:
		system("cls");
		ShowListUsers(vUsers);
		return;

	case enUsersChoise::AddNewuser:
		system("cls");
		AddUsers();
		return;

	case enUsersChoise::DeleteUser:
		system("cls");
		DeleteUsers(vUsers);
		return;

	case enUsersChoise::Updateuser:
		system("cls");
		UpdateUser(vUsers);
		return;

	case enUsersChoise::FindUser:
		system("cls");
		ResultresarchFindUser(vUsers);
		return;

	case enUsersChoise::Mainmenue:
		system("cls");
		StartSystemClients(User);
		return;

	}

}

int ChackNameAndpasswordUser(string Name, string Password, vector<stUsers> vUsers)
{
	int numUser = 0;
	for (stUsers &n : vUsers)
	{
		numUser++;
		if (n.Name == Name && n.Password == Password)

			return numUser ;
	}
	return numUser;
}

void ReadNameUserAndPassword()
{
	while(true){
		string Name = ReadString("Enter Uesr Name? ");
		string Password = ReadString("Enter Password? ");
		vector<stUsers> vUsers = FormFileUsersToVector(UsersFileName);
		int numUser = ChackNameAndpasswordUser(Name, Password, vUsers);

		if (numUser)
		{
			StartSystemClients(vUsers[numUser-1]);
			break;
		}
		else
				cout << "Invaild UserName/Password! \n";
	}

}

void ShowLoginScreen()
{
	cout << "\n-----------------------------------------------\n\n";
	cout << "               Login Screen\n";
	cout << "\n-----------------------------------------------\n";

	ReadNameUserAndPassword();
}

int main() {


	ShowLoginScreen();

	system("pause>0");
	return 0;
}
