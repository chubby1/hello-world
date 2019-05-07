
#include<stdio.h>
#include<mpi.h>
double f(double x)
{
  return (4.0/(1+x*x));
}
double Trap(double a,double b,int n,double h)
{
  double sum=0.0;
  double temp_x=a;
  sum=(f(a)+f(b))/2.0;
  for(int i=1;i<=n-1;i++)
  {
    temp_x+=h;
    sum+=f(temp_x);
  }
  return sum*h;
}

int main()
{
  int my_rank,comm_sz;
  double a=0.0,b=1.0;
  double n=400;
  double h=(b-a)/n;
  double local_a,local_b,local_n,local_total;
  
  
  MPI_Init(NULL,NULL);
  MPI_Comm_size(MPI_COMM_WORLD,&comm_sz);
  MPI_Comm_rank(MPI_COMM_WORLD,&my_rank);
  
  local_n=n/comm_sz;
  local_a=a+my_rank*local_n*h;
  local_b=local_a+local_n*h;
  local_total=Trap(local_a,local_b,local_n,h);
  
  if(my_rank==0)
  {
    double total=local_total;
    for(int i=1;i<comm_sz;i++)
    {
       MPI_Recv(&local_total,1,MPI_DOUBLE,MPI_ANY_SOURCE,0,MPI_COMM_WORLD,MPI_STATUS_IGNORE);
       total+=local_total;
    }
    printf("the π's value is:%lf",total);
  }
  else
  {
    MPI_Send(&local_total,1,MPI_DOUBLE,0,0,MPI_COMM_WORLD);
  }
  MPI_Finalize();
  return 0;
}
