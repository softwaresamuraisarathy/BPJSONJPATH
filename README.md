from itertools import groupby

data = [    [1, 'A', 'John', 'Doe'],
    [2, 'B', 'Jane', 'Doe'],
    [1, 'C', 'John', 'Doe'],
    [2, 'D', 'Jane', 'Doe'],
    [3, 'E', 'Jim', 'Smith']
]

result = []
for key, group in groupby(data, key=lambda x: x[0]):
    group_result = []
    first_name = group[0][2]
    last_name = group[0][3]
    for item in group:
        code = item[1]
        group_result.append(f"code:{code}|firstname:{first_name}|last name:{last_name}")
    result.append([key, '\n'.join(group_result)])

print(result)


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
          'Arr':[
            {'Name1':'123'},
            {'Name2':'456'}
          ],
          'AllStores':{
          'id':111,
          'name':'aaaa',
          'Arr':[
            {'Name1':'123'},
            {'Name2':'456'}
          ],
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

        //      public static string strjson = @"{
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
            //var qry = "$..Products[*].Name";  //List Of JProperty
            //var qry = "$..Products[?(@.Price == 50)]"; //Single Dictionary (JObject)
            //var qry = "$..Products[?(@.Price >= 50)]"; //Multiple Dictionary (JObject)
            //var qry = "$..Manufacturers"; //Single JArray
            //var qry = "$..Stores"; //Multiple Array
            var qry = "$..Arr"; //Multiple Array of Objects
            //var qry = "$"; //All as it is
            //var qry = "$..AllStores"; //Good Compination
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
            DataColumn DCArrayNumber = new DataColumn("ArrayNumber", typeof(string));
            DataColumn DCArrayIndex = new DataColumn("ArrayIndex", typeof(string));
            DataColumn DCName = new DataColumn("Name", typeof(string));
            DataColumn DCValue = new DataColumn("Value", typeof(string));
            DT.Columns.Add(DCArrayNumber);
            DT.Columns.Add(DCArrayIndex);
            DT.Columns.Add(DCName);
            DT.Columns.Add(DCValue);
            DataRow DR ;
            JObject JObjKVP = null;
            int iArrayNumber = -1;
            int iArrayIndex = -1;
            foreach (JToken JT in IEJT)
            {
                iArrayIndex = iArrayIndex + 1;
                if (!JT.HasValues)
                {
                    JObjKVP = JObject.Parse("{" + JT.Parent.ToString() + "}");
                    foreach (var KVP in JObjKVP)
                    {
                        DR = DT.NewRow();
                        DR[DCArrayNumber.ColumnName] = iArrayNumber;
                        DR[DCArrayIndex.ColumnName] = iArrayIndex;
                        DR[DCName.ColumnName] = KVP.Key.ToString();
                        DR[DCValue.ColumnName] = KVP.Value.ToString();
                        DT.Rows.Add(DR);

                        Console.WriteLine(KVP.Key.ToString());
                        Console.WriteLine(KVP.Value.ToString());
                    }
                }
                else if (JT.Type == JTokenType.Array)
                {
                    iArrayNumber = iArrayNumber + 1;
                    iArrayIndex = -1;
                    foreach (var item in JT)
                    {
                        iArrayIndex = iArrayIndex + 1;
                        if (!item.HasValues)
                        {
                            JObjKVP = JObject.Parse("{\"ztempJProperty\":\"" + item.ToString() + "\"}");
                        }
                        else
                        {
                            JObjKVP = (JObject)item;
                        }

                        foreach (var KVP in JObjKVP)
                        {
                            DR = DT.NewRow();
                            DR[DCArrayNumber.ColumnName] = iArrayNumber;
                            DR[DCArrayIndex.ColumnName] = iArrayIndex;
                            DR[DCName.ColumnName] = KVP.Key.ToString();
                            DR[DCValue.ColumnName] = KVP.Value.ToString();
                            DT.Rows.Add(DR);

                            Console.WriteLine(KVP.Key.ToString());
                            Console.WriteLine(KVP.Value.ToString());
                        }

                        JObjKVP = null;
                    }
                    //TempJT = JT.SelectToken("$[0]");

                }
                else 
                {
                    JObjKVP = (JObject)JT;
                    foreach (var KVP in JObjKVP)
                    {
                        DR = DT.NewRow();
                        DR[DCArrayNumber.ColumnName] = iArrayNumber;
                        DR[DCArrayIndex.ColumnName] = iArrayIndex;
                        DR[DCName.ColumnName] = KVP.Key.ToString();
                        DR[DCValue.ColumnName] = KVP.Value.ToString();
                        DT.Rows.Add(DR);

                        Console.WriteLine(KVP.Key.ToString());
                        Console.WriteLine(KVP.Value.ToString());
                    }
                }
                //break;
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
--------------------------------------------------------------JObject JObj = JObject.Parse(strjson);
IEnumerable<JToken> IEJT = JObj.SelectTokens(query);

JToken TempJT = null;

//=================== JSON to Collection Single Row ==========
DataTable DT = new DataTable("NameValue");
DataColumn DCArrayNumber = new DataColumn("ArrayNumber", typeof(string));
DataColumn DCArrayIndex = new DataColumn("ArrayIndex", typeof(string));
DataColumn DCName = new DataColumn("Name", typeof(string));
DataColumn DCValue = new DataColumn("Value", typeof(string));
DT.Columns.Add(DCArrayNumber);
DT.Columns.Add(DCArrayIndex);
DT.Columns.Add(DCName);
DT.Columns.Add(DCValue);
DataRow DR ;
JObject JObjKVP = null;
int iArrayNumber = -1;
int iArrayIndex = -1;
foreach (JToken JT in IEJT)
{
	iArrayIndex = iArrayIndex + 1;
	if (!JT.HasValues)
	{
		JObjKVP = JObject.Parse("{" + JT.Parent.ToString() + "}");
		foreach (var KVP in JObjKVP)
		{
			DR = DT.NewRow();
			DR[DCArrayNumber.ColumnName] = iArrayNumber;
			DR[DCArrayIndex.ColumnName] = iArrayIndex;
			DR[DCName.ColumnName] = KVP.Key.ToString();
			DR[DCValue.ColumnName] = KVP.Value.ToString();
			DT.Rows.Add(DR);

			Console.WriteLine(KVP.Key.ToString());
			Console.WriteLine(KVP.Value.ToString());
		}
	}
	else if (JT.Type == JTokenType.Array)
	{
		iArrayNumber = iArrayNumber + 1;
		iArrayIndex = -1;
		foreach (var item in JT)
		{
			iArrayIndex = iArrayIndex + 1;
			if (!item.HasValues)
			{
				JObjKVP = JObject.Parse("{\"ztempJProperty\":\"" + item.ToString() + "\"}");
			}
			else
			{
				JObjKVP = (JObject)item;
			}

			foreach (var KVP in JObjKVP)
			{
				DR = DT.NewRow();
				DR[DCArrayNumber.ColumnName] = iArrayNumber;
				DR[DCArrayIndex.ColumnName] = iArrayIndex;
				DR[DCName.ColumnName] = KVP.Key.ToString();
				DR[DCValue.ColumnName] = KVP.Value.ToString();
				DT.Rows.Add(DR);

				Console.WriteLine(KVP.Key.ToString());
				Console.WriteLine(KVP.Value.ToString());
			}

			JObjKVP = null;
		}
		//TempJT = JT.SelectToken("$[0]");

	}
	else 
	{
		JObjKVP = (JObject)JT;
		foreach (var KVP in JObjKVP)
		{
			DR = DT.NewRow();
			DR[DCArrayNumber.ColumnName] = iArrayNumber;
			DR[DCArrayIndex.ColumnName] = iArrayIndex;
			DR[DCName.ColumnName] = KVP.Key.ToString();
			DR[DCValue.ColumnName] = KVP.Value.ToString();
			DT.Rows.Add(DR);

			Console.WriteLine(KVP.Key.ToString());
			Console.WriteLine(KVP.Value.ToString());
		}
	}
	//break;
}

resultcollection = DT;
