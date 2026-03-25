use stellar_sdk::{
    client::SyncClient,
    network::Network,
    transaction::{Transaction, TransactionEnvelope},
    assets::{Asset, AssetType},
    payments::Payment,
    keypair::Keypair,
    AccountId,
    Result,
};

pub struct Invoice {
    pub id: String,
    pub amount: u64,
    pub currency: String,
    pub payer: AccountId,
    pub status: String,
}

impl Invoice {
    pub fn new(id: String, amount: u64, currency: String, payer: AccountId) -> Self {
        Invoice {
            id,
            amount,
            currency,
            payer,
            status: "Pending".to_string(),
        }
    }

    pub async fn pay(&self, secret_key: &str, dest_account: AccountId) -> Result<TransactionEnvelope> {
        let client = SyncClient::new("https://horizon-testnet.stellar.org")?;
        
        let keypair = Keypair::from_secret(secret_key)?;

        // Load the account from Stellar network
        let account = client.get_account(&keypair.public_key()).await?;

        // Create a payment operation
        let payment = Payment::new(
            &self.payer,
            &dest_account,
            &Asset::native(),
            self.amount,
        );

        // Create the transaction
        let tx = Transaction::new(vec![payment], &account, 100);

        // Sign the transaction
        let tx_envelope = tx.sign(&keypair);

        // Submit the transaction to the network
        let response = client.submit_transaction(&tx_envelope).await?;

        Ok(response)
    }
}

// Sample usage
#[cfg(test)]
mod tests {
    use super::*;

    #[tokio::test]
    async fn test_invoice_payment() {
        let invoice = Invoice::new("inv_001".to_string(), 1000, "XLM".to_string(), "GDUJRZHEUI7UNITYEMXKZHYYK5Z3BTLF6ERFIRETD34AB2N7UGLEFCIE".to_string());
        let secret_key = "your_secret_key_here"; // Use a valid secret key for testing
        let dest_account = "GDUJRZHEUI7UNITYEMXKZHYYK5Z3BTLF6ERFIRETD34AB2N7UGLEFCIE".to_string(); // Replace with recipient account

        match invoice.pay(secret_key, dest_account).await {
            Ok(response) => println!("Transaction successful: {:?}", response),
            Err(e) => println!("Transaction failed: {:?}", e),
        }
    }
}
``*
