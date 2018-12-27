# BLUETOOTH-BLE-GATT-SERVER-CLIENT
BLUETOOTH BLE GATT SERVER CLIENT EXAMPLE

In MainWindow Constructor create connections:

connect(&m_bleConnection, &BluetoothClient::statusChanged, this, &MainWindow::statusChanged);
connect(&m_bleConnection, SIGNAL(changedState(BluetoothClient::bluetoothleState)),
this,SLOT(changedState(BluetoothClient::bluetoothleState)));

state case in mainwindow:

void MainWindow::changedState(BluetoothClient::bluetoothleState state){

    switch(state){

    case BluetoothClient::Scanning:
    {
        ui->m_cbDevicesFound->clear();

        ui->m_pBConnect->setEnabled(false);
        ui->m_pBSearch->setEnabled(false);
        ui->m_cbDevicesFound->setEnabled(false);

        statusChanged("Searching for low energy devices...");
        break;
    }
    case BluetoothClient::ScanFinished:
    {
        m_bleConnection.getDeviceList(m_qlFoundDevices);

        if(!m_qlFoundDevices.empty()){

            for (int i = 0; i < m_qlFoundDevices.size(); i++)
            {
                ui->m_cbDevicesFound->addItem(m_qlFoundDevices.at(i));
                qDebug() << ui->m_cbDevicesFound->itemText(i);
            }

            /* Initialise Slot startConnect(int) -> button press m_pBConnect */
            connect(this, SIGNAL(connectToDevice(int)),&m_bleConnection,SLOT(startConnect(int)));

            ui->m_pBConnect->setEnabled(true);
            ui->m_pBSearch->setEnabled(true);
            ui->m_cbDevicesFound->setEnabled(true);

            statusChanged("Please select BLE device");
        }
        else
        {
            statusChanged("No Low Energy devices found");
            ui->m_pBSearch->setEnabled(true);
        }

        break;
    }

    case BluetoothClient::Connecting:
    {
        ui->m_pBConnect->setEnabled(false);
        ui->m_pBSearch->setEnabled(false);
        ui->m_cbDevicesFound->setEnabled(false);
        break;
    }
    case BluetoothClient::Connected:
    {
        ui->m_pBConnect->setText("DisConnect");
        connect(&m_bleConnection, SIGNAL(newData(QByteArray)), this, SLOT(DataHandler(QByteArray)));
        ui->m_pBConnect->setEnabled(true);
        ui->m_pBConnect->setText("DisConnect");

        break;
    }
    case BluetoothClient::DisConnected:
    {
        statusChanged("Device disconnected.");
        ui->m_pBConnect->setEnabled(true);
        ui->m_pBConnect->setText("Connect");
        break;
    }
    case BluetoothClient::ServiceFound:
    {
        break;
    }
    case BluetoothClient::AcquireData:
    {
        requestData(mPP);
        requestData(mPI);
        requestData(mPD);
        break;
    }
    default:
        //nothing for now
        break;
    }
}
