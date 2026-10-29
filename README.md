# Chain_of_Responsibility-

1. Суть паттерна
Chain of Responsibility позволяет передавать запрос последовательно по цепочке обработчиков. Каждый обработчик решает, может ли он обработать запрос сам, нужно ли передать его дальше или остановить обработку.

Представь систему обработки заявок в колледже:
- Преподаватель решает обычные вопросы.
- Заведующий отделением разбирает более сложные.
- Директор рассматривает вопросы, требующие высшего уровня полномочий.
Преподавателю не нужно самому знать все правила директора. Он либо решает вопрос, либо передаёт его следующему звену.
Заявка

Например, запрос на покупку оборудования


Преподаватель

Может решить вопрос в пределах своего лимита?


Заведующий отделением

Если полномочий недостаточно — передаёт дальше


Директор

Рассматривает крупную заявку


2. Когда он нужен
Используй этот паттерн, когда:
- заранее неизвестно, какой объект должен обработать запрос;
- несколько обработчиков могут попытаться его обработать;
- хочется добавлять и менять обработчики без переписывания вызывающего кода;
- не хочется создавать длинные конструкции if / else if в одном месте.
Например: обработка HTTP-запросов, фильтры сообщений, проверка платежей, техническая поддержка, логирование.
3. Учебный пример на C#
Создадим цепочку проверки заявок на покупку оборудования. Каждый сотрудник имеет свой лимит согласования.

Структура проекта:
ApprovalChain/
├── Models/
│   └── PurchaseRequest.cs
├── Handlers/
│   ├── ApprovalHandler.cs
│   ├── TeacherHandler.cs
│   ├── DepartmentHeadHandler.cs
│   └── DirectorHandler.cs
└── Program.cs


Файл Models/PurchaseRequest.cs
Модель заявки — информация, которую передают между обработчиками.
namespace ApprovalChain.Models;

public class PurchaseRequest
{
    public string Description { get; }
    public decimal Amount { get; }

    public PurchaseRequest(string description, decimal amount)
    {
        Description = description;
        Amount = amount;
    }
}


Файл Handlers/ApprovalHandler.cs
Базовый обработчик хранит ссылку на следующее звено цепочки.
using ApprovalChain.Models;

namespace ApprovalChain.Handlers;

public abstract class ApprovalHandler
{
    private ApprovalHandler? _nextHandler;

    public ApprovalHandler SetNext(ApprovalHandler handler)
    {
        _nextHandler = handler;
        return handler;
    }

    public void Handle(PurchaseRequest request)
    {
        if (CanApprove(request))
        {
            Approve(request);
            return;
        }

        if (_nextHandler != null)
        {
            _nextHandler.Handle(request);
            return;
        }

        Console.WriteLine(
            $"Заявка «{request.Description}» не согласована: " +
            "нет подходящего обработчика.");
    }

    protected abstract bool CanApprove(PurchaseRequest request);

    protected abstract void Approve(PurchaseRequest request);
}


Разберём ключевые моменты:
- SetNext() задаёт следующий обработчик.
- CanApprove() определяет, подходит ли заявка этому звену.
- Approve() выполняет согласование.
- Handle() решает, обработать запрос или передать дальше.
Файл Handlers/TeacherHandler.cs
using ApprovalChain.Models;

namespace ApprovalChain.Handlers;

public class TeacherHandler : ApprovalHandler
{
    protected override bool CanApprove(PurchaseRequest request)
    {
        return request.Amount <= 5_000;
    }

    protected override void Approve(PurchaseRequest request)
    {
        Console.WriteLine(
            $"Преподаватель согласовал: {request.Description} " +
            $"({request.Amount:N0} руб.)");
    }
}


Файл Handlers/DepartmentHeadHandler.cs
using ApprovalChain.Models;

namespace ApprovalChain.Handlers;

public class DepartmentHeadHandler : ApprovalHandler
{
    protected override bool CanApprove(PurchaseRequest request)
    {
        return request.Amount <= 50_000;
    }

    protected override void Approve(PurchaseRequest request)
    {
        Console.WriteLine(
            $"Заведующий согласовал: {request.Description} " +
            $"({request.Amount:N0} руб.)");
    }
}


Файл Handlers/DirectorHandler.cs
using ApprovalChain.Models;

namespace ApprovalChain.Handlers;

public class DirectorHandler : ApprovalHandler
{
    protected override bool CanApprove(PurchaseRequest request)
    {
        return request.Amount <= 500_000;
    }

    protected override void Approve(PurchaseRequest request)
    {
        Console.WriteLine(
            $"Директор согласовал: {request.Description} " +
            $"({request.Amount:N0} руб.)");
    }
}


Файл Program.cs
Здесь только создание цепочки и отправка заявок. Используем современные top-level statements — без вручную написанного static void Main().
using ApprovalChain.Models;
using ApprovalChain.Handlers;

ApprovalHandler teacher = new TeacherHandler();
ApprovalHandler departmentHead = new DepartmentHeadHandler();
ApprovalHandler director = new DirectorHandler();

teacher.SetNext(departmentHead)
       .SetNext(director);

var requests = new List<PurchaseRequest>
{
    new("Клавиатура", 2_000),
    new("Новый принтер", 25_000),
    new("Компьютерный класс", 350_000),
    new("Серверное оборудование", 700_000)
};

foreach (var request in requests)
{
    teacher.Handle(request);
}


4. Результат выполнения
Преподаватель согласовал: Клавиатура (2 000 руб.)
Заведующий согласовал: Новый принтер (25 000 руб.)
Директор согласовал: Компьютерный класс (350 000 руб.)
Заявка «Серверное оборудование» не согласована:
нет подходящего обработчика.
