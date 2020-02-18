# BPJSONJPATH
BPJSONJPATH
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ConsoleApp2
{
    class Program
    {
        public static string strjson = @"{
          'id':999,
          'Stores': [
            'Lambton Quay1',
            'Willis Street1'
          ],
          'AllStores':{
          'id':111,
          'name':'aaaa',
          'address' : {'Line1':'123 wall st','Line2':'Newyork'},
          'Stores': [
            'Lambton Quay2',
            'Willis Street2'
          ]},
          'Manufacturers': [
            {
              'Name': 'Acme Co',
              'Products': [
                {
                  'Name': 'Anvil',
                  'Price': 50
                },{
                  'Name': '1Anvil',
                  'Price': 501
                }
              ]
            },
            {
              'Name': 'Contoso',
              'Products': [
                {
                  'Name': 'Elbow Grease',
                  'Price': 99.95
                },
                {
                  'Name': 'Headlight Fluid',
                  'Price': 4
                }
              ]
            }
          ]
        }";

        //      public static string strjson = @"{
        //'Name':999}";
        
        static void Main(string[] args)
        {
            //strjson = "";
            if (strjson == null || strjson.Trim() == "")
            {
                strjson = "{}";
            }
            JObject JObj = JObject.Parse(strjson);
            //var qry = "$.id"; //Single JProperty
            //var qry = "$..Products[*].Name";  //List Of JProperty
            //var qry = "$..Products[?(@.Price == 50)]"; //Single Dictionary (JObject)
            //var qry = "$..Products[?(@.Price >= 50)]"; //Multiple Dictionary (JObject)
            //var qry = "$..Manufacturers"; //Single JArray
            //var qry = "$..Stores"; //Multiple Array
            //var qry = "$"; //All as it is
            var qry = "$..AllStores"; //Good Compination
            //var qry = "$..InCorrect"; //Incorrect Qry



            IEnumerable<JToken> IEJT = JObj.SelectTokens(qry);

            //Console.WriteLine(IEJT.ToString());
            JToken TempJT = null;
            string json = "";
            //=================== JSON to Collection ==========
            foreach (JToken JT in IEJT)
            {

                TempJT = JT;
                //Result is single Property. So No Key in JT. Only Value available in result
                if (!JT.HasValues)
                {
                    //TempJT = JT.Parent;
                    json = json + "{" +JT.Parent.ToString() + "}" + ",";
                } 
                else 
                {
                    json = json + TempJT.ToString() + ",";
                        
                }
            }
            if(json != "") 
            {
                json = "{\"" + "ResultArray" + "\"" + ": [" + json.Remove(json.Length - 1, 1) + "]}";
                //json = "[" + json.Remove(json.Length - 1, 1) + "]";
            }
            else
            {
                //json = "{\"" + "ResultArray" + "\"" + ": [{}]}";
                json = "{}";
            }

            JObject JObjResult = JObject.Parse(json);
            //DataTable dt = (DataTable)JsonConvert.DeserializeObject("{[" + json.Remove(json.Length - 1, 1) + "]}", (typeof(DataTable)));
            //=================== JSON to Collection ==========

            //=================== JSON to Collection Single Row ==========
            DataTable DT = new DataTable("NameValue");
            DataColumn DCName = new DataColumn("Name", typeof(string));
            DataColumn DCValue = new DataColumn("Value", typeof(string));
            DT.Columns.Add(DCName);
            DT.Columns.Add(DCValue);
            DataRow DR ;
            JObject JObjKVP = null;
            foreach (JToken JT in IEJT)
            {

                //TempJT = JT;
                //Result is single Property. So No Key in JT. Only Value available in result
                if (!JT.HasValues)
                {
                    JObjKVP = JObject.Parse("{" + JT.Parent.ToString() + "}");
                }
                else if (JT.Type == JTokenType.Array)
                {
                    TempJT = JT.SelectToken("$[0]");
                    if (!TempJT.HasValues)
                    {
                        JObjKVP = JObject.Parse("{\"ztempJProperty\":\"" + TempJT.ToString() + "\"}");
                    }
                    else
                    {
                        JObjKVP = (JObject)TempJT;
                    }                        
                }
                else 
                {
                    JObjKVP = (JObject)JT;
                }

                

                foreach (var KVP in JObjKVP)
                {
                    DR = DT.NewRow();
                    DR[DCName.ColumnName] = KVP.Key.ToString();
                    DR[DCValue.ColumnName] = KVP.Value.ToString();
                    DT.Rows.Add(DR);

                    Console.WriteLine(KVP.Key.ToString());
                    Console.WriteLine(KVP.Value.ToString());
                }
                break;
            }
            //=================== JSON to Collection Single Row ==========
            Console.ReadLine();
        }

        public static DataSet JSON2DS1() {
            DataSet data = JsonConvert.DeserializeObject<DataSet>(strjson);
            return data;
        }
        
        public static DataTable Tabulate(string json)
        {
            var jsonLinq = JObject.Parse(@"{
  'results':
  [
    {
      'Enabled': true,
      'Id': 106,
      'Name': 'item 1',
    },
    {
      'Enabled': false,
      'Id': 107,
      'Name': 'item 2',
      '__metadata': { 'Id': 4013 }
    }
  ]
}");

            // Find the first array using Linq
            var srcArray = jsonLinq.Descendants().Where(d => d is JArray).First();
            var trgArray = new JArray();
            foreach (JObject row in srcArray.Children<JObject>())
            {
                var cleanRow = new JObject();
                foreach (JProperty column in row.Properties())
                {
                    // Only include JValue types
                    if (column.Value is JValue)
                    {
                        cleanRow.Add(column.Name, column.Value);
                    }
                }

                trgArray.Add(cleanRow);
            }

            return JsonConvert.DeserializeObject<DataTable>(trgArray.ToString());
        }
    }
}
-------------------------------------------single-------------------------------------------------
if (strjson == null || strjson.Trim() == "")
{
	strjson = "{}";
}
			
JObject JObj = JObject.Parse(strjson);
IEnumerable<JToken> IEJT = JObj.SelectTokens(query);

JToken TempJT = null;

//=================== JSON to Collection Single Row ==========
DataTable DT = new DataTable("NameValue");
DataColumn DCName = new DataColumn("Name", typeof(string));
DataColumn DCValue = new DataColumn("Value", typeof(string));
DT.Columns.Add(DCName);
DT.Columns.Add(DCValue);
DataRow DR ;
JObject JObjKVP = null;
foreach (JToken JT in IEJT)
{
	//Result is single Property. So No Key in JT. Only Value available in result
	if (!JT.HasValues)
	{
		JObjKVP = JObject.Parse("{" + JT.Parent.ToString() + "}");
	}
	else if (JT.Type == JTokenType.Array)
	{
		TempJT = JT.SelectToken("$[0]");
		if (!TempJT.HasValues)
		{
			JObjKVP = JObject.Parse("{\"ztempJProperty\":\"" + TempJT.ToString() + "\"}");
		}
		else
		{
			JObjKVP = (JObject)TempJT;
		}                        
	}
	else 
	{
		JObjKVP = (JObject)JT;
	}

	

	foreach (var KVP in JObjKVP)
	{
		DR = DT.NewRow();
		DR[DCName.ColumnName] = KVP.Key.ToString();
		DR[DCValue.ColumnName] = KVP.Value.ToString();
		DT.Rows.Add(DR);

		Console.WriteLine(KVP.Key.ToString());
		Console.WriteLine(KVP.Value.ToString());
	}
	break;
}

resultcollection = DT;
-----------------------------------------------------------------------------------------------------
----------------------------------------Get All------------------------------------------------
if (strjson == null || strjson.Trim() == "")
{
	strjson = "{}";
}
JObject JObj = JObject.Parse(strjson);
IEnumerable<JToken> IEJT = JObj.SelectTokens(query);

JToken TempJT = null;
string json = "";
//=================== JSON to Collection ==========
foreach (JToken JT in IEJT)
{
	TempJT = JT;
	//Result is single Property. So No Key in JT. Only Value available in result
	if (!JT.HasValues)
	{
		json = json + "{" +JT.Parent.ToString() + "}" + ",";
	} 
	else 
	{
		json = json + TempJT.ToString() + ",";			
	}
}
if(json != "") 
{
	json = "{\"" + "ResultArray" + "\"" + ": [" + json.Remove(json.Length - 1, 1) + "]}";
}
else
{
	json = "{}";
}

JObject JObjResult = JObject.Parse(json);
JSONResult = json;
-----------------------------------------------------------------------------------------------------

-------------------------------------UnWrapp------------------------------------------------
ResultCollection = (DataTable)InputCollection.Rows[0][0];
ResultCollection = (DataTable)ResultCollection.Rows[0][0];
-----------------------------------------------------------------------------------------------------
