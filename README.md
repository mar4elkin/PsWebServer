# PsWebServer

Civet web server integration plugin for Unreal Engine 5

## HowTo

Web server usage (f.e. in AMyGameMode::BeginPlay()))
```cpp
void AMyGameMode::BeginPlay()
{
	Super::BeginPlay();
	
	auto PlayerController = Cast<APlayerController>(UGameplayStatics::GetPlayerController(this, 0));

	WebServer = NewObject<UPsWebServer>(this);
	WebServer->StartServer();
	
	const auto SomeOAuthHandler = NewObject<USomeOAuthHandler>(this);
	SomeOAuthHandler->PlayerController = PlayerController;
	SomeOAuthHandler->SetHeader(
		TEXT("Content-Type"),
		TEXT("text/plain; charset=utf-8")
	);
	WebServer->AddHandler(SomeOAuthHandler, TEXT("/some_oauth/"));

	PlayerController->OnAuthCompleteEvent.BindLambda([this](FString Response)
	{
		WebServer->StopServer();
	});
}
```

USomeOAuthHandler.h 
```cpp
UCLASS()
class USomeOAuthHandler : public UPsWebServerHandler
{
	GENERATED_BODY()

public:
	virtual void ProcessRequest_Implementation(const FGuid& RequestUniqueId, const FString& RequestData) override;

public:
	UPROPERTY()
	APlayerController* PlayerController = nullptr;
};
```

USomeOAuthHandler.cpp
```cpp
void USomeOAuthHandler::ProcessRequest_Implementation(const FGuid& RequestUniqueId, const FString& RequestData)
{
	if (!RequestData.IsEmpty() && PlayerController)
	{
		PlayerController->OnAuthCompleteEvent.Execute(RequestData);
	}
	
	ProcessRequestFinish(RequestUniqueId, FString::Printf(TEXT("{\"request_data\":%s}"), *RequestData));
	return;
}
```

The user can now access the resource through this link: ```http://127.0.0.1:2050/some_oauth/```
