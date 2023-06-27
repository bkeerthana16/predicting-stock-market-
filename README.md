#module code
import os
import sys


def main():
    """Run administrative tasks."""
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'predicting_stock_markettrends.settings')
    try:
        from django.core.management import execute_from_command_line
    except ImportError as exc:
        raise ImportError(
            "Couldn't import Django. Are you sure it's installed and "
            "available on your PYTHONPATH environment variable? Did you "
            "forget to activate a virtual environment?"
        ) from exc
    execute_from_command_line(sys.argv)


if __name__ == '__main__':
    main()
#remote user
from django.db.models import Count
from django.db.models import Q
from django.shortcuts import render, redirect, get_object_or_404
import datetime
import openpyxl


# Create your views here.
from Remote_User.models import ClientRegister_Model,stock_market_model,predicting_stock_markettrends_model



def login(request):


    if request.method == "POST" and 'submit1' in request.POST:

        username = request.POST.get('username')
        password = request.POST.get('password')
        try:

            enter = ClientRegister_Model.objects.get(username=username, password=password)
            request.session["userid"] = enter.id


            return redirect('Add_DataSet_Details')
        except:
            pass

    return render(request,'RUser/login.html')

def Add_DataSet_Details(request):
    if "GET" == request.method:
        return render(request, 'RUser/Add_DataSet_Details.html', {})
    else:
        excel_file = request.FILES["excel_file"]

        # you may put validations here to check extension or file size

        wb = openpyxl.load_workbook(excel_file)

        # getting all sheets
        sheets = wb.sheetnames
        print(sheets)

        # getting a particular sheet
        worksheet = wb["Sheet1"]
        print(worksheet)

        # getting active sheet
        active_sheet = wb.active
        print(active_sheet)

        # reading a cell
        print(worksheet["A1"].value)

        excel_data = list()
        # iterating over the rows and
        # getting value from each cell in row
        for row in worksheet.iter_rows():
            row_data = list()
            for cell in row:
                row_data.append(str(cell.value))
                print(cell.value)
            excel_data.append(row_data)

            stock_market_model.objects.all().delete()

    for r in range(1, active_sheet.max_row+1):
        stock_market_model.objects.create(
        Company_Name=active_sheet.cell(r, 1).value,
        Company_Category=active_sheet.cell(r, 2).value,
        Opening_Price=active_sheet.cell(r, 3).value,
        Date_Of_Opening=active_sheet.cell(r, 4).value,
        Closing_Price=active_sheet.cell(r, 5).value,
        Date_Of_Closing=active_sheet.cell(r, 6).value,
        volume=active_sheet.cell(r, 7).value,
        Profit='---',
        prices_drop='---',
        Stock_Market_Trends='---',
        Stock_Exchange_By=active_sheet.cell(r, 11).value
        )

    return render(request, 'RUser/Add_DataSet_Details.html', {"excel_data": excel_data})


def Register1(request):

    if request.method == "POST":
        username = request.POST.get('username')
        email = request.POST.get('email')
        password = request.POST.get('password')
        phoneno = request.POST.get('phoneno')
        country = request.POST.get('country')
        state = request.POST.get('state')
        city = request.POST.get('city')
        ClientRegister_Model.objects.create(username=username, email=email, password=password, phoneno=phoneno,
                                            country=country, state=state, city=city)

        return render(request, 'RUser/Register1.html')
    else:

        return render(request,'RUser/Register1.html')


def ViewYourProfile(request):
    userid = request.session['userid']
    obj = ClientRegister_Model.objects.get(id= userid)
    return render(request,'RUser/ViewYourProfile.html',{'object':obj})


def Search_StockMarket_DataSets(request):
    if request.method == "POST":
        kword = request.POST.get('keyword')
        print(kword)
        obj = stock_market_model.objects.all().filter(Company_Name__contains=kword)

        obj1 = stock_market_model.objects.get(Company_Name__contains=kword)
        opening = int(obj1.Opening_Price)
        closing = int(obj1.Closing_Price)
        trends = closing - opening

        if (opening < closing):
            val = 'Profit'
            Stock_Market_Trends = 'Uptrends'
        if (opening > closing):
            val = 'prices drop'
            Stock_Market_Trends = 'downtrends'
        if (opening == closing):
            val = 'Horizontal'
            Stock_Market_Trends = 'HorizontalTrends'

        return render(request, 'RUser/Search_StockMarket_DataSets.html',{'objs': obj,'trends': trends,'val': val,'Stock_Market_Trends': Stock_Market_Trends})
    return render(request, 'RUser/Search_StockMarket_DataSets.html')


def ratings(request,pk):
    vott1, vott, neg = 0, 0, 0
    objs = stock_market_model.objects.get(id=pk)
    unid = objs.id
    vot_count = stock_market_model.objects.all().filter(id=unid)
    for t in vot_count:
        vott = t.ratings
        vott1 = vott + 1
        obj = get_object_or_404(stock_market_model, id=unid)
        obj.ratings = vott1
        obj.save(update_fields=["ratings"])
        return redirect('Add_DataSet_Details')

    return render(request,'RUser/ratings.html',{'objs':vott1})

#server user

from django.db.models import  Count, Avg
from django.shortcuts import render, redirect
from django.db.models import Count
from django.db.models import Q
import datetime


# Create your views here.
from Remote_User.models import ClientRegister_Model,stock_market_model,predicting_stock_markettrends_model


def serviceproviderlogin(request):
    if request.method  == "POST":
        admin = request.POST.get('username')
        password = request.POST.get('password')
        if admin == "SProvider" and password =="SProvider":
            stock_market_model.objects.all().delete()
            predicting_stock_markettrends_model.objects.all().delete()
            return redirect('View_Remote_Users')

    return render(request,'SProvider/serviceproviderlogin.html')


def viewtreandingquestions(request,chart_type):
    dd = {}
    pos,neu,neg =0,0,0
    poss=None
    topic = predicting_stock_markettrends_model.objects.values('ratings').annotate(dcount=Count('ratings')).order_by('-dcount')
    for t in topic:
        topics=t['ratings']
        pos_count=predicting_stock_markettrends_model.objects.filter(topics=topics).values('names').annotate(topiccount=Count('ratings'))
        poss=pos_count
        for pp in pos_count:
            senti= pp['names']
            if senti == 'positive':
                pos= pp['topiccount']
            elif senti == 'negative':
                neg = pp['topiccount']
            elif senti == 'nutral':
                neu = pp['topiccount']
        dd[topics]=[pos,neg,neu]
    return render(request,'SProvider/viewtreandingquestions.html',{'object':topic,'dd':dd,'chart_type':chart_type})

def Search_StockMarket(request): # Search

   if request.method == "POST":
        kword = request.POST.get('keyword')
        print(kword)
        obj = stock_market_model.objects.all().filter(Company_Name__contains=kword)

        obj1 = stock_market_model.objects.get(Company_Name__contains=kword)
        opening=int(obj1.Opening_Price)
        closing=int(obj1.Closing_Price)
        trends=closing-opening

        if(opening<closing):
            val='Profit'
            Stock_Market_Trends='Uptrends'
        if(opening>closing):
            val = 'prices drop'
            Stock_Market_Trends = 'downtrends'
        if (opening == closing):
            val = 'Horizontal'
            Stock_Market_Trends = 'HorizontalTrends'

        return render(request, 'SProvider/Search_StockMarket.html', {'objs': obj,'trends': trends,'val': val,'Stock_Market_Trends': Stock_Market_Trends})

   return render(request, 'SProvider/Search_StockMarket.html')

def View_All_StockMarket_Prediction_Details(request):

    pl=0
    pl1=0
    obj1 = stock_market_model.objects.values(
        'Company_Name',
        'Company_Category',
        'Opening_Price',
        'Date_Of_Opening',
        'Closing_Price',
        'Date_Of_Closing',
        'volume',
        'Profit',
        'prices_drop',
        'Stock_Market_Trends',
        'Stock_Exchange_By')

    predicting_stock_markettrends_model.objects.all().delete()
    for t in obj1:

        Company_Name=t['Company_Name']
        Company_Category=t['Company_Category']
        Opening_Price=int(t['Opening_Price'])
        Date_Of_Opening=t['Date_Of_Opening']
        Closing_Price=int(t['Closing_Price'])
        Date_Of_Closing=t['Date_Of_Closing']
        volume=t['volume']
        Profit=t['Profit']
        prices_drop=t['prices_drop']
        Stock_Market_Trends=t['Stock_Market_Trends']
        Stock_Exchange_By=t['Stock_Exchange_By']

        Total =Closing_Price-Opening_Price


        if (Opening_Price < Closing_Price):
            val = 'Profit'
            Stock_Market_Trends = 'Up trends'
            finalstr=val+':'+Stock_Market_Trends
        if (Opening_Price > Closing_Price):
            val = 'prices drop'
            Stock_Market_Trends = 'down trends'
            finalstr = val + ':' + Stock_Market_Trends
        if (Opening_Price == Closing_Price):
            val = 'Horizontal'
            Stock_Market_Trends = 'Horizontal Trends'
            finalstr = val + ':' + Stock_Market_Trends
        if (Total > 0):
            pl = Total
            predicting_stock_markettrends_model.objects.create(names=Company_Name, Company_Category=Company_Category,
                                                               Opening_Price=Opening_Price,
                                                               Date_Of_Opening=Date_Of_Opening,
                                                               Closing_Price=Closing_Price,
                                                               Date_Of_Closing=Date_Of_Closing, volume=volume,
                                                               Profit=pl, prices_drop=0, Stock_Market_Trends=finalstr,
                                                               Stock_Exchange_By=Stock_Exchange_By)
        if (Total < 0):
            pl1 = Total

            predicting_stock_markettrends_model.objects.create(names=Company_Name, Company_Category=Company_Category,
                                                           Opening_Price=Opening_Price, Date_Of_Opening=Date_Of_Opening,
                                                           Closing_Price=Closing_Price, Date_Of_Closing=Date_Of_Closing,
                                                           volume=volume, Profit=0, prices_drop=pl1,
                                                           Stock_Market_Trends=finalstr,
                                                           Stock_Exchange_By=Stock_Exchange_By)
        if (Total == 0):
            pl1 = Total

            predicting_stock_markettrends_model.objects.create(names=Company_Name, Company_Category=Company_Category,
                                                           Opening_Price=Opening_Price, Date_Of_Opening=Date_Of_Opening,
                                                           Closing_Price=Closing_Price, Date_Of_Closing=Date_Of_Closing,
                                                           volume=volume, Profit=0, prices_drop=0,
                                                           Stock_Market_Trends=finalstr,
                                                           Stock_Exchange_By=Stock_Exchange_By)

    obj = predicting_stock_markettrends_model.objects.all()
    return render(request, 'SProvider/View_All_StockMarket_Prediction_Details.html', {'objs': obj})

def View_Remote_Users(request):
    obj=ClientRegister_Model.objects.all()
    return render(request,'SProvider/View_Remote_Users.html',{'objects':obj})

def ViewTrendings(request):
    topic = predicting_stock_markettrends_model.objects.values('topics').annotate(dcount=Count('topics')).order_by('-dcount')
    return  render(request,'SProvider/ViewTrendings.html',{'objects':topic})

def negativechart(request,chart_type):
    dd = {}
    pos, neu, neg = 0, 0, 0
    poss = None
    topic = predicting_stock_markettrends_model.objects.values('ratings').annotate(dcount=Count('ratings')).order_by('-dcount')
    for t in topic:
        topics = t['ratings']
        pos_count = predicting_stock_markettrends_model.objects.filter(topics=topics).values('names').annotate(topiccount=Count('ratings'))
        poss = pos_count
        for pp in pos_count:
            senti = pp['names']
            if senti == 'positive':
                pos = pp['topiccount']
            elif senti == 'negative':
                neg = pp['topiccount']
            elif senti == 'nutral':
                neu = pp['topiccount']
        dd[topics] = [pos, neg, neu]
    return render(request,'SProvider/negativechart.html',{'object':topic,'dd':dd,'chart_type':chart_type})


def charts(request,chart_type):
    chart1 = predicting_stock_markettrends_model.objects.values('names').annotate(dcount=Avg('Profit'))
    return render(request,"SProvider/charts.html", {'form':chart1, 'chart_type':chart_type})

def charts1(request,chart_type):
    chart1 = predicting_stock_markettrends_model.objects.values('names').annotate(dcount=Avg('prices_drop'))
    return render(request,"SProvider/charts1.html", {'form':chart1, 'chart_type':chart_type})

def View_StockMarket_Details(request):
    obj =stock_market_model.objects.all()
    return render(request, 'SProvider/View_StockMarket_Details.html', {'list_objects': obj})

def likeschart(request,like_chart):
    charts =predicting_stock_markettrends_model.objects.values('names').annotate(dcount=Avg('Profit'))
    return render(request,"SProvider/likeschart.html", {'form':charts, 'like_chart':like_chart})

def View_StockMarketUpDown(request):
    obj = predicting_stock_markettrends_model.objects.all()
    return render(request, 'SProvider/View_StockMarketUpDown.html', {'objs': obj})









