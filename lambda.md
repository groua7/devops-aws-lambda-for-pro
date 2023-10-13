import boto3

def get_lambda_functions_for_instance(ec2_instance_id, lambda_client):
    # Get all Lambda functions
    lambda_functions = lambda_client.list_functions()['Functions']

    # Find Lambda functions associated with the given EC2 instance ID
    associated_functions = []

    for function in lambda_functions:
        if 'VpcConfig' in function and 'SubnetIds' in function['VpcConfig']:
            for subnet_id in function['VpcConfig']['SubnetIds']:
                # Extract EC2 instance ID from the Subnet ID
                instance_id_from_subnet = get_instance_id_from_subnet(subnet_id)

                # Check if the instance ID matches
                if instance_id_from_subnet == ec2_instance_id:
                    associated_functions.append(function['FunctionName'])

    return associated_functions

def get_instance_id_from_subnet(subnet_id):
    # Extract the instance ID from the subnet ID
    parts = subnet_id.split('-')
    return parts[-1]

def main():
    # Specify your AWS region
    region = 'your_aws_region'

    # Specify your EC2 instance ID
    ec2_instance_id = 'your_ec2_instance_id'

    # Create EC2 and Lambda clients
    ec2_client = boto3.client('ec2', region_name=region)
    lambda_client = boto3.client('lambda', region_name=region)

    # Get information about the specified EC2 instance
    instance_info = ec2_client.describe_instances(InstanceIds=[ec2_instance_id])

    # Extract the VPC ID of the instance
    vpc_id = instance_info['Reservations'][0]['Instances'][0]['VpcId']

    # Print EC2 instance details
    print(f"EC2 Instance ID: {ec2_instance_id}")
    print(f"VPC ID: {vpc_id}")

    # Get Lambda functions associated with the EC2 instance
    associated_functions = get_lambda_functions_for_instance(ec2_instance_id, lambda_client)

    if associated_functions:
        print("Lambda Functions associated with the EC2 instance:")
        for function_name in associated_functions:
            print(f"  - {function_name}")
    else:
        print("No Lambda Functions associated with the EC2 instance.")

if __name__ == "__main__":
    main()
