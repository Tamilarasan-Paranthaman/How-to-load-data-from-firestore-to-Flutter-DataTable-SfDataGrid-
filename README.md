# How-to-load-data-from-firestore-to-Flutter-DataTable-SfDataGrid-

Load data from the [Firestore](https://firebase.google.com/docs/firestore) to the [Flutter DataGrid](https://www.syncfusion.com/flutter-widgets/flutter-datagrid) widget by fetching the data with the required collection name from Firestore. Then, create the rows for the DataGrid from document data.

The following steps explain how to load the data from the Firestore database for Flutter DataGrid:

## STEP 1:
To get started with Firestore, refer to this [link](https://firebase.google.com/docs/firestore) and create the database in Firestore. In this KB, we have explained using the Firestore data as an example. We will not provide Firebase access files and a key. 

## STEP 2: 
To fetch the data from the Firestore, you need to add the following package in the dependencies of pubspec.yaml.

```dart
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^1.20.1
  cloud_firestore: ^3.4.4

```

## STEP 3: 
Import the following library into the flutter application:

```dart
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:syncfusion_flutter_datagrid/datagrid.dart';
```
## STEP 4: 
Initialize the [SfDataGrid](https://pub.dev/documentation/syncfusion_flutter_datagrid/latest/datagrid/SfDataGrid-class.html) with all the required details. Fetch the data from the Firestore database by passing the required collection name.  In the following sample, we have shown how to perform CRUD operations from the Firestore. If you add the new document to the collection or update/delete the existing document in that collection, we have done the respective operations in the DataGrid based on the DocumentChangeType(added/ removed/ modified) in the document. 

```dart
class SfDataGridDemo extends StatefulWidget {
  const SfDataGridDemo({Key? key}) : super(key: key);

  @override
  SfDataGridDemoState createState() => SfDataGridDemoState();
}

class SfDataGridDemoState extends State<SfDataGridDemo> {
  late EmployeeDataSource employeeDataSource;
  List<Employee> employeeData = [];

  final getData = FirebaseFirestore.instance.collection('Employee').snapshots();

  Widget _buildDataGrid() {
    return StreamBuilder(
      stream: getData,
      builder: (BuildContext context, AsyncSnapshot<QuerySnapshot> snapshot) {
        if (snapshot.hasData) {
          if (employeeData.isNotEmpty) {
            
            // to update the value changed at runtime
            getDataGridRowFromDataBase(DocumentChange<Object?> data) {
              return DataGridRow(cells: [
                DataGridCell<String>(columnName: 'id', value: data.doc['id']),
                DataGridCell<String>(
                    columnName: 'name', value: data.doc['name']),
                DataGridCell<String>(
                    columnName: 'designation', value: data.doc['designation']),
                DataGridCell<String>(
                    columnName: 'salary', value: data.doc['salary'].toString()),
              ]);
            }

            for (var data in snapshot.data!.docChanges) {
              if (data.type == DocumentChangeType.modified) {
                if (data.oldIndex == data.newIndex) {
                  employeeDataSource.dataGridRows[data.oldIndex] =
                      getDataGridRowFromDataBase(data);
                }
                employeeDataSource.updateDataGridSource();
              } else if (data.type == DocumentChangeType.added) {
                employeeDataSource.dataGridRows
                    .add(getDataGridRowFromDataBase(data));
                employeeDataSource.updateDataGridSource();
              } else if (data.type == DocumentChangeType.removed) {
                employeeDataSource.dataGridRows.removeAt(data.oldIndex);
                employeeDataSource.updateDataGridSource();
              }
            }
          } else {
            for (var data in snapshot.data!.docs) {
              employeeData.add(Employee(
                  id: data['id'],
                  name: data['name'],
                  designation: data['designation'],
                  salary: data['salary'].toString()));
            }
            employeeDataSource = EmployeeDataSource(employeeData);
          }

          return SfDataGrid(
            source: employeeDataSource,
            columns: getColumns,
            columnWidthMode: ColumnWidthMode.fill,
          );
        } else {
          return const Center(
            child: CircularProgressIndicator(),
          );
        }
      },
    );
  }
  @override
  void initState() {
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Syncfusion Flutter Datagrid FireStore Demo'),
      ),
      body: _buildDataGrid(),
    );
  }
}

```

## STEP 5: 
Create a data source class that extends with the [DataGridSource](https://pub.dev/documentation/syncfusion_flutter_datagrid/latest/datagrid/DataGridSource-class.html) for mapping data to the [SfDataGrid](https://pub.dev/documentation/syncfusion_flutter_datagrid/latest/datagrid/SfDataGrid-class.html).

```dart
class EmployeeDataSource extends DataGridSource {
  EmployeeDataSource(this.employeeData) {
    _buildDataRow();
  }

  List<DataGridRow> dataGridRows = [];
  List<Employee> employeeData;

  void _buildDataRow() {
    dataGridRows = employeeData
        .map<DataGridRow>((e) => DataGridRow(cells: [
              DataGridCell<String>(columnName: 'id', value: e.id),
              DataGridCell<String>(columnName: 'name', value: e.name),
              DataGridCell<String>(
                  columnName: 'designation', value: e.designation),
              DataGridCell<String>(columnName: 'salary', value: e.salary),
            ]))
        .toList();
  }

  @override
  List<DataGridRow> get rows => dataGridRows;

  @override
  DataGridRowAdapter buildRow(
    DataGridRow row,
  ) {
    return DataGridRowAdapter(
        cells: row.getCells().map<Widget>((e) {
      return Container(
        alignment: Alignment.center,
        padding: const EdgeInsets.all(8.0),
        child: Text(e.value.toString()),
      );
    }).toList());
  }

  void updateDataGridSource() {
    notifyListeners();
  }
}

```